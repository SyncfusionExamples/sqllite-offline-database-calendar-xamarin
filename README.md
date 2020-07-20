# How to load the data from SQLite offline database into SfCalendar?

SfCalendar allows you to bind the data from local database by using SQLite.

Refer the following link to know how to create SQLite connection,

[https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/databases](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/databases)

You can also refer the following article

https://www.syncfusion.com/kb/9380/how-to-load-the-data-from-sqlite-offline-database-into-sfcalendar

**C#**

Custom appointment class defined.

``` c#
   public class ViewModel
    {
        SQLiteConnection database;
        public CalendarEventCollection Meetings { get; set; }

        public ViewModel()
        {
            //Create connection
            database = DependencyService.Get<ISQLite>().GetConnection();

            //Create table
            database.CreateTable<AppointmentDB>();

            var currentDate = DateTime.Now.Date;
            database.Query<AppointmentDB>("DELETE From AppointmentDB");

            //Insert data in to table 
            database.Query<AppointmentDB>("INSERT INTO AppointmentDB (Subject,StartTime,EndTime,AllDay,Color)values ('Yoga Therapy','" + currentDate.AddHours(1).ToString() + "', '" + currentDate.AddHours(2).ToString() + "','false','#ff0000')");
            database.Query<AppointmentDB>("INSERT INTO AppointmentDB (Subject,StartTime,EndTime,AllDay,Color)values ('Client Meeting','" + currentDate.ToString() + "', '" + currentDate.AddHours(1).ToString() + "','true','#0000ff')");
            database.Query<AppointmentDB>("INSERT INTO AppointmentDB (Subject,StartTime,EndTime,AllDay,Color)values ('Client Meeting','" + currentDate.AddHours(5).ToString() + "', '" + currentDate.AddHours(7).ToString() + "','false','#ffa500')");
            AddAppointments();
        }

        /// <summary>
        /// Creates meetings and stores in a collection.  
        /// </summary>
        private void AddAppointments()
        {
            var table = (from i in database.Table<AppointmentDB>() select i);
            Meetings = new CalendarEventCollection();
            foreach (var order in table)
            {
                var EventName = order.Subject;
                var From = DateTime.Parse(order.StartTime);
                var To = DateTime.Parse(order.EndTime);
                var AllDay = Convert.ToBoolean(order.AllDay);

                Meetings.Add(new CalendarInlineEvent()
                {
                    Subject = order.Subject,
                    StartTime = DateTime.Parse(order.StartTime),
                    EndTime = DateTime.Parse(order.EndTime),
                    Color = Color.FromHex(order.Color),
                    IsAllDay = Convert.ToBoolean(order.AllDay),
                });
            }

        }
    }
```

**C#**

Since SQLite doesn&#39;t support for DateTime type, we have used it as string type and convert into DateTime while adding items to local collection from DB. SQLite data table is created which is used to populate the data and then store them as CalendarEventCollection.

``` c#
    /// <summary>   
    /// Represents custom data properties.   
    /// </summary>  
    public class AppointmentDB
    {
        public string Subject { get; set; }
        public string StartTime { get; set; }
        public string EndTime { get; set; }
        public string AllDay { get; set; }
        public string Color { get; set; }
    }
```

**XAML**

Refer the following code to bind the data from SQLite to SfCalendar by using **DataSource** Property.

``` xml
<syncfusion:SfCalendar x:Name="calendar" ShowInlineEvents="true"
        DataSource = "{Binding Meetings}">
        <syncfusion:SfCalendar.BindingContext>
            <local:ViewModel/>
        </syncfusion:SfCalendar.BindingContext>
    </syncfusion:SfCalendar>
```
