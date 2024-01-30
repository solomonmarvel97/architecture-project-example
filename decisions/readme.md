# Architectural Decision Guide

## Objective
The objective of this architectural decision guide is to provide a structured approach to implementing multiple logins and session management in an existing application using MSSQL Server as the database. The focus is on ensuring smooth user experience, comprehensive session tracking, and database integrity.

## 1. Database Schema Update

Database Name: BBGCorporateInternetBankingDBTest

### 1.1 Concurrent Session Table
```
- **Table Name:** [dbo].[AdminActiveSessionLog]

  - [id] [int] IDENTITY(1,1) NOT NULL,
  - [LoginName] [varchar](50) NOT NULL,
  - [IsActive] [bit] NOT NULL DEFAULT 1,
  - [LastUpdated] [smalldatetime] NOT NULL,

  - **Constraints:**
    - PRIMARY KEY ([SessionId])
```

### 1.2 Existing Session Table Update

```
- **Table Name:** [dbo].[AdminActivityLogs]

  TABLE [dbo].[AdminActivityLogs](
	[ActivityLogId] [bigint] IDENTITY(1,1) NOT NULL,
	[CustomerId] [varchar](50) NOT NULL,
	[LoginName] [varchar](50) NOT NULL,
	[ActivityCategory] [varchar](50) NOT NULL,
	[ActivityDetails] [varchar](200) NOT NULL,
	[LogDate] [smalldatetime] NOT NULL,
	[LoginComputerName] [varchar](200) NOT NULL,
	[LoginComputerIP] [varchar](80) NOT NULL,
CONSTRAINT [PK_AdminActivityLogs] PRIMARY KEY CLUSTERED

  - **Constraints:**
    - PRIMARY KEY ([ActivityLogId])
```

## 2. Session Management Implementation

### 2.1 Login Process

- Upon successful login, create a new entry in the [AdminActiveSessionLog] table with the user's login name, setting IsActive to 1, and recording the current timestamp as LastLoginTime.

### 2.2 Active Sessions

- Implement a mechanism to regularly update the [AdminActiveSessionLog] table, refreshing the LastLoginTime for active sessions to indicate continuous user activity.

### 2.3 Session Expiry Duration

- Periodically check the [AdminActiveSessionLog] table to identify inactive sessions. Set IsActive to 0 for sessions where the LastLoginTime exceeds the defined session expiry duration (e.g., 5 minutes).

### 2.4 Unexpected Closure Handling

- Implement a cleanup process that runs periodically to identify sessions where unexpected closure occurred. If the LastLoginTime is not updated within the expected timeframe, set IsActive to 0 for those sessions.

## 3. Logging

### 3.1 Session Logging

- Continue using the existing logging mechanism for user activity. Ensure that login, logout, and session update activities are appropriately logged in [dbo].[AdminActivityLogs].

## 4. Application Logic

### 4.1 Login Interception

- Modify the login process to check the [AdminActiveSessionLog] table for existing active sessions. If an active session is found, decide whether to terminate the existing session or deny the new login attempt based on business rules.

### 4.2 User Logout

- Provide a mechanism for users to manually log out. On manual logout, update IsActive to 0 in the [AdminActiveSessionLog] table.

## 5. Exception Handling

- Implement robust exception handling mechanisms to address unexpected errors during the session management and login processes, ensuring that the system maintains its integrity even in error scenarios.

## 6. Testing

- Conduct thorough testing of the implemented session management features, including multiple logins, session expiry, unexpected closure handling, and manual logout.

## 7. Documentation

- Update system documentation to include details of the new [AdminActiveSessionLog] table, modifications to the existing tables, and the implemented session management features.

## Conclusion

This architectural decision guide outlines the steps and considerations for implementing multiple logins and session management in the existing application. It ensures a comprehensive approach to tracking user sessions, handling unexpected closures, and maintaining database integrity. The proposed solution aims to enhance user experience and security within the application.
