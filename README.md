# ts-logger-system

var SPREADSHEET_ID = "1eS9Ll_YJuJco34v5e72Jd8A2Mo-6z64yWOMW7NNSUSA"; 
var financialSheetName = 'Sheet1';

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(15000); 

  try {
    var doc = SpreadsheetApp.openById(SPREADSHEET_ID);
    
    // لقط البيانات بأي شكل قادمة بيه (Parameter أو JSON)
    var data = e.parameter;
    if (e.postData && e.postData.contents) {
      try {
        var json = JSON.parse(e.postData.contents);
        if (json) { data = json; }
      } catch(err) {}
    }

    // تأكيد لقط المتغيرات
    var reportType = data.reportType || e.parameter.reportType;

    if (reportType === 'FINANCIAL_LOG') {
      var sheet = doc.getSheetByName(financialSheetName);
      
      if (!sheet) {
        sheet = doc.insertSheet(financialSheetName);
        sheet.appendRow([
          "timestamp", "empId", "empName", "empRole", 
          "shiftNum", "checkIn", "checkOut", "exactHours", 
          "hourlyRate", "todayEarnings", "daysWorked", "totalMonthEarnings"
        ]);
      }

      // رص الـ 11 خانة بالملي
      var rowData = [
        new Date(), 
        data.empId || e.parameter.empId || '',
        data.empName || e.parameter.empName || '',
        data.empRole || e.parameter.empRole || '',
        data.shiftNum || e.parameter.shiftNum || '',
        data.checkIn || e.parameter.checkIn || '',
        data.checkOut || e.parameter.checkOut || '',
        data.exactHours || e.parameter.exactHours || '',
        data.hourlyRate || e.parameter.hourlyRate || '',
        data.todayEarnings || e.parameter.todayEarnings || '',
        data.daysWorked || e.parameter.daysWorked || '',
        data.totalMonthEarnings || e.parameter.totalMonthEarnings || ''
      ];

      sheet.appendRow(rowData);

      return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
    }
    
    return ContentService.createTextOutput("No Action").setMimeType(ContentService.MimeType.TEXT);
  }
  catch (err) {
    return ContentService.createTextOutput("Error: " + err.toString()).setMimeType(ContentService.MimeType.TEXT);
  }
  finally {
    lock.releaseLock(); 
  }
}

// دالة الـ doGet لضمان عدم حدوث مشاكل في الشيت
function doGet(e) {
  return ContentService.createTextOutput("Script is running").setMimeType(ContentService.MimeType.TEXT);
}