# CPS510 Assignment 5: Running the Hotel Menu from a Mac

This guide explains how to upload the hotel project, connect to TMU's Moon Linux server, verify an Oracle login, and run the assignment menu from macOS Terminal.

## 1. Understand the two connections

There are two separate logins:

| Connection | Purpose | Password |
|---|---|---|
| SSH to Moon | Opens a terminal on the school's Linux computer | CS/Linux account password |
| SQL*Plus to Oracle | Connects to the database containing the hotel tables | Oracle database password used in SQL Developer |

A successful SSH login does not automatically log in to Oracle. A menu asking for an Oracle username and password is only collecting those details; it does not confirm that the database connection succeeded.

Mac Terminal and Windows MobaXterm can both connect to Moon. After SSH login, commands run on the same school Linux environment, regardless of the laptop's operating system.

## 2. Connection settings

These settings come from the course handout, `2-connect-Oracle-23ai-instructions-through-ssh-new.pdf`.

| Setting | Value |
|---|---|
| SSH server | `moon.cs.torontomu.ca` |
| SSH port | `22` |
| Oracle host | `orsrv1.cs.torontomu.ca` |
| Oracle port | `1521` |
| Oracle service name | `orcldb` |
| Oracle command on Moon | `sqlplus64` |

The SSH server and Oracle database server are different machines. The database uses **`SERVICE_NAME=orcldb`**, not `SID=orcldb`.

Replace `YOUR_CS_USERNAME` in every command below with the actual CompSci username. Do not type that placeholder literally. Use the account whose Oracle tables are intended for the project. The username is not a full email address.

Use the course's VPN instructions when required for off-campus access. The supplied handout lists Tunnelblick for Mac.

## 3. Prepare the project files

Use the latest corrected `menu.sh`, including the service-name and formatting-path fixes. Keep the complete project folder together.

The important paths are:

| Path inside the project | Purpose |
|---|---|
| `menu.sh` | Main shell menu |
| `sql/format.sql` | Shared report formatting |
| `sql/create_tables.sql` | Creates tables and views |
| `sql/populate_tables.sql` | Adds sample records |
| `sql/view_tables.sql` | Displays table contents |
| `sql/simple_queries.sql` | Runs Q1-Q8 |
| `sql/view_queries.sql` | Runs Q9-Q15 |
| `sql/advanced/` | Contains Q16-Q23 |

For the commands below, the project folder is named **`hotel_a5`** and is in the Mac's **Downloads** folder. Move the complete folder there, or adjust the local folder path in the upload command.

## 4. Upload the folder from the Mac to Moon

**Run this section in the local Mac Terminal, before logging into Moon.**

1. Press **Command + Space**.
2. Type **Terminal** and press Enter.
3. Run each command below, pressing Enter after each line:

```bash
cd ~/Downloads
scp -r hotel_a5 YOUR_CS_USERNAME@moon.cs.torontomu.ca:~/
```

Enter the CS/Linux account password if asked. Wait for the transfer to finish and the terminal prompt to return.

This copies the complete folder to `~/hotel_a5` in the school account. It does not create or change Oracle tables. Re-uploading files updates the server copy of the project.

If the folder is already uploaded and current, continue to the next section.

## 5. Log into Moon

In the local Mac Terminal, run:

```bash
ssh YOUR_CS_USERNAME@moon.cs.torontomu.ca
```

Enter the CS/Linux account password when prompted. Follow the school's server-identity verification instructions if a first-connection prompt appears.

Password characters may not appear while typing. Enter the password normally and press Enter.

After login, confirm the operating system:

```bash
uname -s
```

- **`Linux`**: the terminal is running commands in a Linux environment. After the SSH login above, this is the Moon session.
- **`Darwin`**: commands are still running locally on the Mac. Complete the SSH login before continuing.

All remaining shell commands in this guide run **on Moon**, unless a section explicitly says otherwise.

## 6. Test Oracle directly before using the menu

First check that the school client is available:

```bash
command -v sqlplus64
```

This should print the location of the program. If nothing is printed, confirm the SSH session is on Moon. If it is, check the course's Oracle environment instructions or ask the TA about the client setup.

Next, run this entire command as one line. Replace `YOUR_CS_USERNAME` first:

```bash
sqlplus64 -L 'YOUR_CS_USERNAME@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=orsrv1.cs.torontomu.ca)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=orcldb)))'
```

Use ordinary straight quotation marks, as shown in the code block. Enter the **Oracle password used in SQL Developer** when prompted. It may differ from the SSH password.

If the connection succeeds, a `SQL>` prompt appears. At that prompt, run:

```sql
SELECT USER FROM DUAL;
```

If Oracle returns the account's username, the database connection is confirmed.

An optional check for the project data is:

```sql
SELECT COUNT(*) AS GuestCount FROM GUEST;
```

The original sample data contains five guests. A missing-table error here means the Oracle account does not have access to that table; it does not mean the SSH login failed.

Return to the Moon shell before running the menu:

```sql
EXIT
```

Do not type shell commands such as `cd`, `sed`, or `bash menu.sh` at the `SQL>` prompt.

## 7. Start the menu on Moon

Run:

```bash
cd ~/hotel_a5
ls
```

The listing should include `menu.sh` and the `sql` folder.

Then run this entire line:

```bash
DB_HOST=orsrv1.cs.torontomu.ca DB_PORT=1521 DB_SID=orcldb DB_SERVICE=orcldb bash menu.sh
```

Both database-name variables are supplied for compatibility with earlier menu versions. The corrected script uses `DB_SERVICE` and sends `SERVICE_NAME=orcldb` to Oracle.

Enter the Oracle username and password if prompted. The menu heading should show `orsrv1.cs.torontomu.ca` as the server.

Editing `menu.sh` on the local Mac does not update the file already uploaded to Moon. Either upload the changed file or edit the copy inside the Moon session.

## 8. Check existing data and run the advanced reports

If A3 and A4 have already been completed in this Oracle account:

1. Choose **4** from the main menu to view table contents.
2. Press Enter when asked to return to the menu.
3. Choose **7** to open the advanced-query menu.
4. Choose an individual report, or **A** to run all advanced reports.

The advanced-menu choices are:

| Choice | Query | Main concept |
|---|---|---|
| 1 | Q16: Contact directory | `UNION` |
| 2 | Q17: Rooms never booked | `MINUS` |
| 3 | Q18: Returning guests | `INTERSECT` |
| 4 | Q19: Payment statistics | Aggregates, `STDDEV`, `ROLLUP` |
| 5 | Q20: Revenue by room type | Joins and grouping |
| 6 | Q21: Monthly booking summary | Grouping by month |
| 7 | Q22: Staff service quality | Joins and `HAVING` |
| 8 | Q23: Above-average spending guests | Aggregate subquery |
| A | All advanced reports | Runs Q16-Q23 |
| B | Back | Returns to the main menu |

For A5, capture at least five advanced-query results covering the required concepts. For example, advanced choices **1, 2, 3, 4, and 5** cover set operations, joins, statistical/aggregate functions, and grouping.

On a Mac, press **Command + Shift + 4** and drag around a report to take a screenshot. Include the report title, headings, and result rows. Take fresh screenshots after resolving errors.

The menu also saves output in the project's `output/` folder on Moon. A saved output file can contain errors, so inspect it before using it as evidence in the report.

## 9. Initial setup when this Oracle account has no project data

Each Oracle account has its own tables. Tables created under another teammate's username do not automatically appear under a different account.

- If this account has no project tables, main-menu **2** creates the tables and views, then **3** adds the sample data.
- If the tables already contain the sample records, use **4** to inspect them and proceed to the reports.
- Running population again can produce duplicate-key errors.
- Options **1** and **8** delete the current project tables. Option 8 rebuilds the sample database after dropping it. Use them only when deliberately resetting the project data for a demonstration.

Review the actual SQL output for errors; a menu message or an output filename is not proof that every command succeeded.

## 10. Common errors

| Message or symptom | Meaning and next check |
|---|---|
| `ORA-12545` | The target host cannot be reached or resolved. Check that the menu shows `orsrv1.cs.torontomu.ca`, not a placeholder such as `your.host`. |
| `ORA-12505` | The connection request contains an unrecognized SID. Use the corrected `SERVICE_NAME=orcldb` setting. |
| `ORA-12514` | The listener does not recognize the requested service. Check the host, port, and service against the current course handout. |
| `ORA-01017` | Oracle rejected the login credentials. Check the Oracle username and current database password, not just the SSH password. |
| `ORA-28000` | The Oracle account is locked. Contact the appropriate course support/TA rather than repeatedly retrying. |
| `SP2-0640: Not connected` | No active Oracle connection exists. Find the first preceding `ORA-...` error and resolve it first. |
| `ORA-00942` while viewing a project table | The account cannot access that table. Confirm the Oracle username and whether A3 was run in that account. |
| `ORA-00955` while creating tables | An object with that name already exists. Check the existing project before attempting creation again. |
| `ORA-00001` while populating | A key value already exists. The sample records may already have been inserted. |
| `sqlplus64: command not found` | The command is unavailable in the current environment. Confirm the Moon login and the course's Oracle client setup. |
| `menu.sh: No such file or directory` | The terminal is in the wrong folder, or the project was not uploaded there. Use `pwd` and `ls` to check. |
| `SP2-0310` for `../format.sql` | The formatting include cannot be resolved. Check the layout and use the corrected menu described below. |

### An older menu still uses SID

From the project folder on Moon, the following changes the connection keyword and keeps a backup named `menu.sh.bak`:

```bash
sed -i.bak 's/(SID=/(SERVICE_NAME=/g' menu.sh
grep 'CONNECT_STR=' menu.sh
```

The connection string must contain `(SERVICE_NAME=...)`. A shell variable named `DB_SID` may remain in an older script; the important part sent to Oracle is the `SERVICE_NAME` keyword and the value `orcldb`.

Restart using the full command in section 7. This edit fixes the connection keyword; it does not add the separate formatting-path fix to an old menu.

### The formatting file exists but cannot be opened

From `~/hotel_a5`, check:

```bash
ls sql/format.sql sql/advanced/q16_union.sql
```

Keep `format.sql` at `sql/format.sql`. The advanced scripts contain `@@../format.sql`.
The corrected `menu.sh` starts SQL*Plus from the selected SQL file's folder, allowing this parent-folder path to resolve. Upload that corrected menu over the old remote copy, then exit and restart the program.

If the issue persists, record `pwd`, the two file paths above, and the exact `SP2-0310` message. This is a file-loading issue; it is separate from Oracle authentication.

## 11. Leave the sessions

| Current location | Exit action |
|---|---|
| Advanced-query menu | Press Enter if paused, then choose **B** |
| Main menu | Choose **E** |
| Oracle `SQL>` prompt | Type `EXIT` |
| Moon shell | Type `exit` to return to the local Mac Terminal |

When troubleshooting, keep the exact first Oracle error and the command that produced it. Do not include passwords in screenshots, reports, or messages.

## References


- [Apple: Connect to servers in Terminal on Mac](https://support.apple.com/guide/terminal/connect-to-servers-trml1018/mac)
- [Oracle: SQL*Plus Quick Start](https://docs.oracle.com/en/database/oracle/oracle-database/26/sqpug/SQL-Plus-quick-start.html)
- [Oracle: Using Scripts in SQL*Plus](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqpug/using-scripts-in-SQL-Plus.html)

These instructions describe the supplied project and course connection settings. Successful execution still needs to be confirmed in the student's own Moon and Oracle accounts.
