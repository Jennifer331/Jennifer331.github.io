---
layout: "post"
title: "数据库操作"
---

# <font color="#e3796b">Content Provider VS Direct Database Access</font>

# <font color="#e3796b">Content Provider</font>
把逻辑拆分成三部分
* <font color="#d1559b">\*\*\*Contract</font>
参考[CalendarContract]()

__信息__
* <font color="#d1559b">\*\*\*Provider</font>
参考[MediaProvider](https://android.googlesource.com/platform/packages/providers/MediaProvider/+/master/src/com/android/providers/media/MediaProvider.java)

__逻辑__
* <font color="#d1559b">\*\*\*DatabaseHelper</font>
参考[CalendarDatabaseHelper](https://android.googlesource.com/platform/packages/providers/CalendarProvider/+/master/src/com/android/providers/calendar/CalendarDatabaseHelper.java)

__操作__
