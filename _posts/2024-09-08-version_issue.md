---
title: Spring Boot 2.x - > 3.x 업그레이드 시 겪었던 SQL 관련 문제
author: gyuhwan
date: 2024-09-09 15:40:00 +0800
categories: [Spring, Issue]
tags: [Spring]
---

업무로 인해 Spring Boot 2.x에서 3.x로 업그레이드하면서 여러 문제를 겪었다. 그 중 첫 번째로 발생한 에러를 살펴보자.

```sql
Caused by: java.sql.SQLSyntaxErrorException: (conn=3) You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '' at line 1
	at org.mariadb.jdbc.export.ExceptionFactory.createException(ExceptionFactory.java:282)
	at org.mariadb.jdbc.export.ExceptionFactory.create(ExceptionFactory.java:370)
	at org.mariadb.jdbc.message.ClientMessage.readPacket(ClientMessage.java:134)
	....
```

이 에러는 SQL 문법상의 오류를 나타낸다. 문제의 원인은 SQL 문의 separator를 설정하지 않아서 발생한 것이었고 `spring.sql.init.separator`에 사용된 separator를 직접 설정함으로써 해결할 수 있었다. 아래와 같은 설정으로 변경한 후 실행하니 에러가 사라졌다.

```yaml
spring:
  sql:
    init:
      encoding: "UTF-8"
      schema-locations: "classpath*:db/schema.sql"
      data-locations: "classpath*:db/data.sql"
      separator: ^;
```

모든 동작이 정상적으로 작동하였고, 스크립트를 다시 확인해보니 잘못된 오타를 발견했다. 사용된 스크립트는 `data.sql`과 `schema.sql` 두 가지다.

**schema.sql**

```sql
CREATE TABLE IF NOT EXISTS `imsi`.`imsi_send` (
.....중략
COMMENT = '임시 전송 관리 테이블';

CREATE TABLE IF NOT EXISTS `imsi2`.`imsi_send` (
.....중략
COMMENT = '임시2 전송 관리 테이블'^;                     <---------------------- ?

CREATE TABLE IF NOT EXISTS `imsi3`.`imsi_send` (
.....중략
COMMENT = '임시3 전송 관리 테이블';
....
```

**data.sql**

```sql
IF NOT EXISTS (SELECT NULL from INFORMATION_SCHEMA.COLUMNS
...중략
END IF^;

IF EXISTS (SELECT NULL from INFORMATION_SCHEMA.COLUMNS
...중략
END IF^;

IF NOT EXISTS (SELECT NULL from INFORMATION_SCHEMA.COLUMNS
...중략
END IF^;
....
```

여기서 흥미로운 점은 사용된 separator가 ‘;’와 ‘^;’ 두 가지라는 것이다. 하지만 `schema.sql`을 살펴보면 대부분의 SQL 문에서는 ‘;’를 사용하고, 단 한 개의 SQL 문만 ‘^;’를 사용하고 있었다. 이는 분명 오타로 보였고 그래서 ‘^;’ →  ‘;’로 수정하여 스크립트를 통일했다.

오타를 수정하고 깔끔하게 정리된 스크립트를 보면서 뿌듯한 마음으로 다시 실행했으나, 또다시 에러가 발생했다.

```yaml
Caused by: java.sql.SQLSyntaxErrorException: (conn=3) You have an error in your SQL syntax; 
		check the manual that corresponds to your MariaDB server version for the right syntax to use near '' at line 1
at org.mariadb.jdbc.export.ExceptionFactory.createException(ExceptionFactory.java:282)
at org.mariadb.jdbc.export.ExceptionFactory.create(ExceptionFactory.java:370)
at org.mariadb.jdbc.message.ClientMessage.readPacket(ClientMessage.java:134)
at org.mariadb.jdbc.client.impl.StandardClient.readPacket(StandardClient.java:872)
at org.mariadb.jdbc.client.impl.StandardClient.readResults(StandardClient.java:811)
at org.mariadb.jdbc.client.impl.StandardClient.readResponse(StandardClient.java:730)
at org.mariadb.jdbc.client.impl.StandardClient.execute(StandardClient.java:654)
at org.mariadb.jdbc.Statement.executeInternal(Statement.java:957)
at org.mariadb.jdbc.Statement.execute(Statement.java:1083)
at org.mariadb.jdbc.Statement.execute(Statement.java:474)
at com.zaxxer.hikari.pool.ProxyStatement.execute(ProxyStatement.java:94)
at com.zaxxer.hikari.pool.HikariProxyStatement.execute(HikariProxyStatement.java)
at org.springframework.jdbc.datasource.init.ScriptUtils.executeSqlScript(ScriptUtils.java:261)
	....
```

**결국 다시 `schema.sql`에서 한 곳만 ‘^;’로 변경하니 정상적으로 실행되었다.. 도무지 이해할 수 없는 상황이었고 정확한 원인을 파악하기 위해 디버깅을 시작했다.**

**SQL을 작성하고 실제로 실행되는 부분은 spring jdbc 안에 있는 [ScriptUtils.java](http://scriptutils.java/)의 executeSqlScript 메서드이다.**

```jsx
public static void executeSqlScript(Connection connection, EncodedResource resource, boolean continueOnError,
	boolean ignoreFailedDrops, String[] commentPrefixes, @Nullable String separator,
	String blockCommentStartDelimiter, String blockCommentEndDelimiter) throws ScriptException {

try {
	if (logger.isDebugEnabled()) {
		logger.debug("Executing SQL script from " + resource);
	}
	long startTime = System.currentTimeMillis();

	String script;
	try {
		script = readScript(resource, separator, commentPrefixes, blockCommentEndDelimiter);  ◀–––––– 중요
	}
	catch (IOException ex) {
		throw new CannotReadScriptException(resource, ex);
	}

	if (separator == null) {
		separator = DEFAULT_STATEMENT_SEPARATOR;
	}
	if (!EOF_STATEMENT_SEPARATOR.equals(separator) &&                     
			!containsStatementSeparator(resource, script, separator, commentPrefixes,  ◀––––––– 중요
				blockCommentStartDelimiter, blockCommentEndDelimiter)) {
		separator = FALLBACK_STATEMENT_SEPARATOR;
	}

	List<String> statements = new ArrayList<>();
	splitSqlScript(resource, script, separator, commentPrefixes, blockCommentStartDelimiter,
			blockCommentEndDelimiter, statements);

	int stmtNumber = 0;
	Statement stmt = connection.createStatement();
	try {
		for (String statement : statements) {
			stmtNumber++;
			try {
				stmt.execute(statement);
				int rowsAffected = stmt.getUpdateCount();
				if (logger.isDebugEnabled()) {
					logger.debug(rowsAffected + " returned as update count for SQL: " + statement);
					SQLWarning warningToLog = stmt.getWarnings();
					while (warningToLog != null) {
						logger.debug("SQLWarning ignored: SQL state '" + warningToLog.getSQLState() +
								"', error code '" + warningToLog.getErrorCode() +
								"', message [" + warningToLog.getMessage() + "]");
						warningToLog = warningToLog.getNextWarning();
					}
				}
	.....
}
```

여기서 `readScript`를 통해 `schema.sql` 스크립트를 읽어 String에 담는다. 매개변수로 받은 separator가 null이라면 기본적으로 ‘;’ 값으로 할당된다. 하지만 yaml에 설정한 값은 ‘^;’이었고, 결국 separator에 대한 값은 containsStatementSeparator 메서드에 의해 결정된다.

```java
private static boolean containsStatementSeparator(@Nullable EncodedResource resource, String script,
		String separator, String[] commentPrefixes, String blockCommentStartDelimiter,
		String blockCommentEndDelimiter) throws ScriptException {

	boolean inSingleQuote = false;
	boolean inDoubleQuote = false;
	boolean inEscape = false;

	for (int i = 0; i < script.length(); i++) {
		char c = script.charAt(i);
		if (inEscape) {
			inEscape = false;
			continue;
		}
		// MySQL style escapes
		if (c == '\\') {
			inEscape = true;
			continue;
		}
		if (!inDoubleQuote && (c == '\'')) {
			inSingleQuote = !inSingleQuote;
		}
		else if (!inSingleQuote && (c == '"')) {
			inDoubleQuote = !inDoubleQuote;
		}
		if (!inSingleQuote && !inDoubleQuote) {
			if (script.startsWith(separator, i)) {
				return true;
			}
			else if (startsWithAny(script, commentPrefixes, i)) {
				// Skip over any content from the start of the comment to the EOL
				int indexOfNextNewline = script.indexOf('\n', i);
				if (indexOfNextNewline > i) {
					i = indexOfNextNewline;
					continue;
				}
				else {
					// If there's no EOL, we must be at the end of the script, so stop here.
					break;
				}
			}
			else if (script.startsWith(blockCommentStartDelimiter, i)) {
				// Skip over any block comments
				int indexOfCommentEnd = script.indexOf(blockCommentEndDelimiter, i);
				if (indexOfCommentEnd > i) {
					i = indexOfCommentEnd + blockCommentEndDelimiter.length() - 1;
					continue;
				}
				else {
					throw new ScriptParseException(
							"Missing block comment end delimiter: " + blockCommentEndDelimiter, resource);
				}
			}
		}
	}

	return false;
}

```

이 메서드는 매개변수로 넘어온 separator가 해당 스크립트에 존재하는지 확인한다. 만약 존재하면 true, 그렇지 않으면 false를 반환한다. 즉, 현재 스크립트에 ‘^;’가 없으면 false를 반환하게 된다.

이 경우 false를 반환하면 separator는 기본적으로 ‘\n’으로 할당된다.

```java
public static final String FALLBACK_STATEMENT_SEPARATOR = "\n";

...

public static void executeSqlScript(Connection connection, EncodedResource resource, boolean continueOnError,
		boolean ignoreFailedDrops, String[] commentPrefixes, @Nullable String separator,
		String blockCommentStartDelimiter, String blockCommentEndDelimiter) throws ScriptException {
		...
	if (!EOF_STATEMENT_SEPARATOR.equals(separator) &&
			!containsStatementSeparator(resource, script, separator, commentPrefixes,
				blockCommentStartDelimiter, blockCommentEndDelimiter)) {
		separator = FALLBACK_STATEMENT_SEPARATOR;
	}	
}
```

따라서 스크립트 내에 ‘^;’가 하나도 없으면 자동으로 ‘\n’으로 설정되어 에러가 계속 발생했던거였다. 이제야 스크립트에 단 한 개의 ‘^;’가 존재했던 이유를 이해할 수 있었다.

처음부터 스크립트의 구분자를 통일했으면 좋았을 텐데, 그 이유에 대해서는 아직도 궁금하다.