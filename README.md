# NOTE: THIS SCRIPT DOES NOT WORK AT THE MOMENT CHECK ISSUES
# gmail-toggl-responder
This is a **Google Apps Script** that's designed to work within your Google Drive without deploying as an add-on or web app.<br>It automatically responds to your client/employer's hours request with a CSV containing your toggl tasks and the total time for the requested date range.

## Change Log
- 5/23/15 - Fixed rounding issue in millisecondsToHMS that caused 00:00:60 to not transform to 00:01:00
- 5/12/15 - Added option (`SHOW_TOTALS_PER_PROJECT`) to show total hours per project

## How It Works
1. Your client/employer sends you an email with the subject containing the specified keyword (default: 'hours') and a date range in the format 'Month dd' or 'Mon dd'. Single digits ('d') are also acceptable.  
2. Valid: `Hours for March 23 to April 5`, `jan 10 february 1 hours`  
3. Invalid: `March 23 to April 5`, `Hours January 10`
4. The script runs on a time-driven trigger set in the Google Apps Script editor, its the button with the chat bubble clock thing
5. The script parses the subjects of **unread** emails to find a thread the specified keyword, a date range, and **without** a `HRespProcessed` tag.
6. The script then fetches a detailed breakdown of tasks within the date range from toggl
7. Data is then converted to CSV and sent to your client/employer as an attachment
8. Finally the email thread is marked `HRespProcessed`

## Usage
To use this script certian variables must be set within the script. Most are already set.
- **required**
  - `API_TOKEN`
  - `WORKSPACE_ID`
  - `KEYWORD`

- optional
  - `CLIENT_IDS`
  - `CSV_HEADERS`
  - `MESSAGE_BODY`
  - `ENTRY_TO_CSV`
  - `SHOW_TOTALS_PER_PROJECT`

### API_TOKEN
Your `API_TOKEN` can be found by going to the toggl app, clicking _My Profile_ under your name at the top right hand corner, and its on the bottom of the page.

### WORKSPACE_ID
Your `WORKSPACE_ID` can be found by going to any one of the reports for your desired workspace. The Id can be found in the address bar as a series of numbers.  

> toggl.com/app/reports/weekly/**123456**/period/prevWeek/billable/both

In this case `WORKSPACE_ID = '123456'`

### KEYWORD
`KEYWORD` is **not** case-sensitive. It can be almost anything; a word or a phrase.

### CLIENT_IDS
`CLIENT_IDS` can be obtained by applying a client filter to a report. The URL will look somthing like:  

> toggl.com/app/reports/weekly/123456/period/prevWeek/clients/**98765432**/billable/both

Thus `CLIENT_IDS = '98765432'`

More clients can be added by adding a comma like so **WITHOUT A SPACE**:

`CLIENT_IDS = '98765432,36273283'`

### CSV_HEADERS
`CSV_HEADERS` is exactly the same as the first row of a CSV file, its the headers delimited by a comma. They can be named anything.

**Important!**: The last two columns will always be `,,Total Duration`. If these are removed there won't be any problems. However the generated CSV file will have a timestamp in an unlabeled column on the first data row

### MESSAGE_BODY
The message to send with the CSV attachment

### ENTRY_TO_CSV
`ENTRY_TO_CSV` isn't as straight forward as the rest of the variables.
- It is a dictionary where the keys are keys for toggl's raw data found here, [LINK](https://github.com/toggl/toggl_api_docs/blob/master/reports/detailed.md#data-array).
- The values of the dictionary are functions used to modify the raw data from toggl. I've defined some already to use.  
- For example, in the link above the data 'dur' is given in milliseconds and the function `millisecondsToHMS` converts that to a string in the format HH:MM:SS for use in the CSV. Thus `ENTRY_TO_CSV` is:

```JavaScript
ENTRY_TO_CSV = {
  'res': millisecondsToHMS
};
```

- `ENTRY_TO_CSV` matches up to `CSV_HEADERS`. Total Duration is automatically appended using `millisecondsToHMS`. For example, if `CSV_HEADERS = 'Start,Description,Duration,,Total Duration'` then in order and following the toggl data keys

```JavaScript
ENTRY_TO_CSV = {
  'start': dateToString,
  'description: doNothing,
  'dur': millisecondsToHMS
};
```

### SHOW_TOTALS_PER_PROJECT
set to `true` to show total hours per project
