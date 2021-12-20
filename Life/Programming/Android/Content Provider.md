Content Provider is an abstraction that allows to receive data from other application, share data with other applications and manage acces to data, stored by itself.
Content Provider provides secure way to share data with other application through standard interface. Also it acts like data interface, decoupling storage implementation from usage by other applications.
Content provider also is used when:  
You want to implement custom search suggestions in your application
You need to use a content provider to expose your application data to widgets
You want to copy and paste complex data or files from your application to other applications
Content provider allows to control permissions for accessing data.
You can use a content provider to abstract away the details for accessing different data sources in your application. For example, your application might store structured records in a SQLite database, as well as video and audio files. You can use a content provider to access all of this data, if you implement this development pattern in your application.
Content provider share data in a SQL format(using tables)