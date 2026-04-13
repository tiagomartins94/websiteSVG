@less(
  formatDateTime(convertTimeZone(item()?['Created'],'UTC','GMT Standard Time'),'yyyy-MM-dd'),
  formatDateTime(addDays(convertTimeZone(utcNow(),'UTC','GMT Standard Time'),-7),'yyyy-MM-dd')
)


_api/web/RecycleBin('@{outputs('RecycleBinItemId')}')/deleteObject()
