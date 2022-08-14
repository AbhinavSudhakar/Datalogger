# Datalogger
Logging the latitude and longitude of a location on google sheets and pointing out the location on a map.
Steps:
1. Copy datalogger_arduino onto Arduino IDE
2. Create a google sheet
3. Copy the sheet ID and paste it under sheet_id in datalogger_arduino
4. Create a new project on google apps script, and name the project as the same name as the google sheet.
5. Paste datalogger_appsscript onto Code.gs of Google Apps Script
6. Create a new deployment, deploy as web app, give the required permissions and copy the deployment ID onto datalogger_arduino under the variable GAS_ID
7. Now, your project is ready to go. Plug in the prototype, upload the code and test it out or yourself!
