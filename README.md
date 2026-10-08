# SQL Server Error Log Monitoring and Alerting

## Objective

The SQL Server Error Log Monitoring and Alerting project aimed to automatically monitor the SQL Server error log for important errors and security related events within a defined time period. The stored procedure checks the most recent 15 minutes of the SQL Server error log, identifies messages containing errors, failed login attempts, or I/O-related issues, and sends an automated HTML formatted email alert through SQL Server Database Mail. This hands on project provided practical experience with SQL Server error log monitoring, `xp_readerrorlog`, temporary tables, dynamic SQL, error detection, T-SQL automation, and Database Mail notifications.

### Skills Learned

* Monitoring the SQL Server error log.
* Reading SQL Server error log information using `xp_readerrorlog`.
* Filtering error log messages based on specific keywords.
* Detecting SQL Server errors and failed login attempts.
* Identifying I/O related error messages.
* Monitoring recent SQL Server activity using date and time filters.
* Using temporary tables to store error log information.
* Using dynamic SQL to execute `xp_readerrorlog`.
* Generating HTML formatted email reports using T-SQL.
* Configuring SQL Server Database Mail for automated alerts.
* Performing proactive SQL Server error and security monitoring.

### Tools Used

* **Microsoft SQL Server 2016** for database administration and error monitoring.
* **SQL Server Management Studio (SSMS)** for T-SQL development and troubleshooting.
* **T-SQL** for stored procedure development and automation.
* **`xp_readerrorlog`** for reading SQL Server error log entries.
* **Temporary Tables** for storing and filtering error log information.
* **SQL Server Database Mail** for automated email notifications.

## Steps

Below are the key steps taken in the SQL Server error log monitoring and alerting process:

### 1. Define the Error Log Monitoring Time Window

The stored procedure monitors the SQL Server error log for the previous 15 minutes.

The monitoring period is defined using:

```sql id="2v0k0h"
DECLARE
@from_date datetime = DATEADD(MINUTE,-15,GETDATE()),
@to_date datetime = GETDATE()
```

This allows the procedure to focus on recent error log activity instead of processing the entire SQL Server error log.

*Ref 1: Stored Procedure*

`![Error Log Monitoring Time Window](screenshots/01-error-log-time-window.png)`
`![Error Log Monitoring Time Window](screenshots/01-error-log-time-window.png)`

### 2. Create a Temporary Error Log Table

A temporary table is created to store the SQL Server error log information retrieved by `xp_readerrorlog`.

```sql id="f4rj7y"
CREATE TABLE #errorLog
(
    LogDate DATETIME,
    ProcessInfo VARCHAR(64),
    Text VARCHAR(MAX)
)
```

The table stores:

* Log date and time
* Process information
* Error log message text

### 3. Read the SQL Server Error Log

The stored procedure uses `xp_readerrorlog` to retrieve error log entries within the specified time range.

The query is dynamically created using the start and end times:

```sql id="u1f5hr"
SET @query = 'EXEC xp_ReadErrorLog 0, 1, NULL, NULL,
'''+CONVERT(VARCHAR(MAX),@from_date)+''',
'''+CONVERT(VARCHAR(MAX),@to_date)+''''
```

The results are then inserted into the temporary table:

```sql id="f0jvwu"
INSERT INTO #errorLog
EXEC (@query)
```

This allows the procedure to analyze recent SQL Server error log activity.

*Ref 3: SQL Server Error Log Retrieval*
This screenshot shows the execution of `xp_readerrorlog` and the retrieval of recent SQL Server error log entries.

`![SQL Server Error Log Retrieval](screenshots/03-read-error-log.png)`

### 4. Detect Important Error Log Messages

The procedure searches the retrieved error log entries for specific keywords:

```sql id="4jy3w6"
WHERE Text LIKE '%error%'
   OR Text LIKE '%Login failed%'
   OR Text LIKE '%I/O%'
```

The monitoring process focuses on three important categories:

* **Error messages** — identifies SQL Server errors.
* **Login failed** — identifies failed authentication attempts.
* **I/O** — identifies potential storage or disk-related issues.

The number of matching records is stored in the `@count` variable.

*Ref 4.1: Login Error Created*
`![Error Detection and Filtering](screenshots/04-error-detection.png)`

*Ref 4.2: Error Detection and Filtering*
This screenshot shows the filtering logic used to identify SQL Server errors, failed login attempts, and I/O related messages.

`![Error Detection and Filtering](screenshots/04-error-detection.png)`

### 5. Generate an HTML Error Report

If matching errors are detected, the procedure converts the results into HTML table rows using:

```sql id="z9kq3e"
FOR XML PATH('tr'), ELEMENTS
```

The report includes:

* Log Date
* Process Information
* Error Log Text

The results are then combined with HTML and CSS formatting to create a readable email report.

### 6. Send Automated SQL Server Error Alert

When matching error log entries are detected, the procedure creates a server-specific email subject:

```sql id="4aqp4v"
SET @sub = @server_name + ': SQL Server Error Log detected'
```

The alert is then sent using SQL Server Database Mail:

```sql id="q8w4d4c"
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'SQL Server Mail Profile',
    @body = @body,
    @body_format = 'HTML',
    @recipients = 'vishnuprasad19931994@gmail.com',
    @subject = @sub;
```

This provides an automated notification whenever a relevant error is detected in the SQL Server error log.

*Ref 6: SQL Server Error Email Alert*
This screenshot shows the Database Mail configuration and automated error notification.

`![SQL Server Error Email Alert](screenshots/06-error-email-alert.png)`

### 7. Receive and Review the Error Log Alert

The received email contains the detected SQL Server error log entries in an HTML-formatted table.

The DBA can review:

* When the error occurred.
* Which SQL Server process generated the message.
* The complete error log message.
* Whether the event was related to authentication, I/O, or another SQL Server error.

The server name is also included in the email subject to help identify which SQL Server instance generated the alert.

*Ref 7: Received SQL Server Error Alert*
This screenshot shows the received email containing the detected SQL Server error log entries.

`![Received SQL Server Error Alert](screenshots/07-received-error-alert.png)`

### 8. Investigate and Troubleshoot Detected Errors

The detected error log messages were reviewed to determine the potential cause and impact of each event.

The investigation can include:

* Reviewing the complete SQL Server error message.
* Checking failed login attempts and authentication issues.
* Investigating I/O-related errors and storage problems.
* Checking SQL Server services and database availability.
* Reviewing related database or application activity.
* Checking whether the error is recurring.
* Investigating the affected database, login, or SQL Server component.
* Taking corrective action based on the type of error detected.

After corrective action, the SQL Server error log was monitored again to verify whether the issue had been resolved and whether similar errors continued to occur.

