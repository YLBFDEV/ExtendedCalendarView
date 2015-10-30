ExtendedCalendarView
====================
![ScreenShot](https://raw.githubusercontent.com/tyczj/ExtendedCalendarView/master/ExtendedCalendarView/Screenshot_2014-04-04-18-11-16.png)

Currently there is no easy way of showing a calendar with the ability to display events on days, ExtendedCalendarView is meant to solve that problem.

Usage
=====

simply declare it in your layout

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent" >
    
    <com.tyczj.extendedcalendarview.ExtendedCalendarView 
        android:id="@+id/calendar"
        android:layout_height="match_parent"
        android:layout_width="match_parent"/>
    
    </RelativeLayout>
    
get the view like you normally would

    ExtendedCalendarView calendar = (ExtendedCalendarView)findViewById(R.id.calendar);

Calendar Content Provider
=========================

All events are stored in a content provider for easy access and the ability to have other app hook into your calendar if you choose. make sure you declare the content provider in your manifest

    <provider
        android:name="com.tyczj.extendedcalendarview.CalendarProvider"
        android:authorities="com.tyczj.extendedcalendarview.calendarprovider" />
                
if you dont want other apps to have access to your database make you add this attribute to the provider 
    
    android:permission="signature"    
    
Current database columns

    id - database id of the event
    event (Text) - name of the event
    location (Text) - where the event is
    description (Text) - information about the event
    start (Integer) - when the event starts
    end (Integer) - when the event ends
    start_day (Integer) - julian start day
    end_day (Integer) - julian end day
    color (Integer) - the color of the event

Adding Events
=============

To add an event to the content provider you need the start time, end time, julian start day and julian end day. For now you will have to implement your own way to get all the information but eventually in the future I may create one that you can just call and use.

    ContentValues values = new ContentValues();
		values.put(CalendarProvider.COLOR, Event.COLOR_RED);
		values.put(CalendarProvider.DESCRIPTION, "Some Description");
		values.put(CalendarProvider.LOCATION, "Some location);
		values.put(CalendarProvider.EVENT, "Event name);
			
		Calendar cal = Calendar.getInstance();
			
		cal.set(startDayYear, startDayMonth, startDayDay, startTimeHour, startTimeMin);
		values.put(CalendarProvider.START, cal.getTimeInMillis());
		values.put(CalendarProvider.START_DAY, julianDay);
		TimeZone tz = TimeZone.getDefault();
			
		cal.set(endDayYear, endDayMonth, endDayDay, endTimeHour, endTimeMin);
		int endDayJulian = Time.getJulianDay(cal.getTimeInMillis(), TimeUnit.MILLISECONDS.toSeconds(tz.getOffset(cal.getTimeInMillis())));
			
		values.put(CalendarProvider.END, cal.getTimeInMillis());
		values.put(CalendarProvider.END_DAY, endDayJulian);

		Uri uri = getContentResolver().insert(CalendarProvider.CONTENT_URI, values);
		
julian start day is generated for you when the month is built so all you would have to do it call day.getStartDay() on the day and it will give you the julian day


i show the informations in a listview below the calendar.
==========

like this.
[Show event information with ExtendedCalendarView](http://stackoverflow.com/questions/26143047/show-event-information-with-extendedcalendarview)

![带日程](http://i.stack.imgur.com/0NxSK.png)

```java

public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_location_site_calendar);
        this.ctx = this;

        this.extendedCalendarView = (ExtendedCalendarView) findViewById(R.id.extendedCalendarView_addLocationSiteCalendar_CALENDAR);
        this.listViewCalendar = (ListView) findViewById(R.id.listView_addLocationSiteCalendar_CALENDARLIST);
        //Disable Scrolling
        this.listViewCalendar.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                return true;
            }
        });

        this.extendedCalendarView.setGesture(ExtendedCalendarView.LEFT_RIGHT_GESTURE);
        addEvent();//test
        addEvent2();//test

        initExtras(savedInstanceState);
        initListener();
    }


private void initListener() {
        extendedCalendarView.setOnDayClickListener(new ExtendedCalendarView.OnDayClickListener() {
            @Override
            public void onDayClicked(AdapterView<?> adapter, View view, int position, long id, Day day) {
                ArrayList<HashMap<Integer, String>> eventList = new ArrayList<HashMap<Integer, String>>();
                for (Event e : day.getEvents()) {
                    HashMap<Integer, String> event = new HashMap<Integer, String>();
                    event.put(NumericValues.LISTROW_ID_IMG_T_ST_IDC_KEY_TITLE, e.getTitle());
                    event.put(NumericValues.LISTROW_ID_IMG_T_ST_IDC_KEY_SUBTITLE, e.getDescription());
                    event.put(NumericValues.LISTROW_ID_IMG_T_ST_IDC_KEY_INDICATOR, e.getStartDate("hh:mm") + " - " + e.getEndDate("hh:mm"));
                    eventList.add(event);
                }
                CalendarListViewAdapter listAdapter = new CalendarListViewAdapter(_this, eventList);
                listViewCalendar.setAdapter(listAdapter);
                ListViewUtil.setListViewHeightBasedOnChildren(listViewCalendar);
            }
        });
    }
```

```xml
<ScrollView
    android:layout_width="fill_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content">

       <com.tyczj.extendedcalendarview.ExtendedCalendarView
            android:id="@+id/extendedCalendarView_addLocationSiteCalendar_CALENDAR"
            android:layout_height="350dp"
            android:layout_width="match_parent" />
        <ListView
            android:id="@+id/listView_addLocationSiteCalendar_CALENDARLIST"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:divider="@drawable/list_divider"
            android:dividerHeight="1px"
            android:listSelector="@drawable/contacts_list_selector" />
    </LinearLayout>
</ScrollView>
```
