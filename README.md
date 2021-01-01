# How to Convert Google Sheets to JSON to HTML

## 1) Google Sheets to JSON

1. In Google Sheets, click _Tools -> Script Editor_
1. Copy this code into the file _Code.gs_ (if necessary, change the sheet name in line 4):
```javascript
function doGet(request) {
  
  // read rows and headers from the named sheet
  var spreadsheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Sheet1');
  var rows = spreadsheet.getRange(1, 1,spreadsheet.getLastRow(), spreadsheet.getLastColumn()).getValues();
  var headers = rows[0];
  
  // this loop populates the "result" array with objects 
  var result = [];
  rows.forEach((row, rowIndex) => {
    if (rowIndex > 0) { 
      var item = new Object();
      row.forEach((cell, cellIndex) => item[headers[cellIndex]] = cell);
      result.push(item);           
    }
  });
  
  // return the JSON as argument of the "prefix"
  if (request.parameters.prefix)
    return ContentService.createTextOutput(request.parameters.prefix + '(' + JSON.stringify(result) + ')')
             .setMimeType(ContentService.MimeType.JAVASCRIPT);

  // if no prefix is defined, return a simple JSON document
  return ContentService.createTextOutput(JSON.stringify(result)).setMimeType(ContentService.MimeType.JSON);
} 
```
3. Click "Publish -> Deploy as web app" and follow the workflow until you hit "Deploy" and gotten this dialog.
1. Follow the link, it should display your sheet as an JSON



## 2) JSON to HTML

If you consume JSON outside the browser, you are golden now - you can use the URL as is. Inside the browser, you will get "CORS header 'Access-Control-Allow-Origin' missing" or an variant of that. That's why you need the JavaScript return in line 20 - it calls a JavaScript function with the JSON as an argument.

1. Make sure that the setup works with this (it should show you an dialog with `[object Object], [object Object], ...`):
```html
<script src="https://script.google.com/macros/<your-id>/exec?prefix=alert"></script>
```
2. Then you can this snippet, which calls the `writeTable()` function  (instead of the built-in `alert()` function) to populate the table from your JSON objects:
```html
<table id="table"></table>

<script>
function writeTable(rows) {
    var tableHtml = "";

    // Create table headers
    tableHtml += '<tr>';	
    var headers = Object.keys(rows[0]);
    headers.forEach((header, headerIndex) => tableHtml += '<th>' + header + '</th>');
    tableHtml += '</tr>';	
  
    // Create table data
    rows.forEach((row, rowIndex) => {
        tableHtml += '<tr>';
        headers.forEach((header, headerIndex) => tableHtml += '<td>' + row[header] + '</td>');
        tableHtml += '</tr>'; 
    });
        
    // Set the generated HTML 
    document.getElementById("table").innerHTML = tableHtml;
}
</script>

<script src="https://script.google.com/macros/<your-id>/exec?prefix=writeTable"></script>
```



## 3) Customize

This is the basic setup, so you might need to do:

- add styling
- add code to access other sheets
- add support for empty JSON documents
