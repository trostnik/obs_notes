Content Provider is an abstraction that allows to receive data from other application, share data with other applications and manage acces to data, stored by itself.

Content Provider provides secure way to share data with other application through standard interface. Also it acts like data interface, decoupling storage implementation from usage by other applications.

Content provider also is used when:  
- You want to implement custom search suggestions in your application
- You need to use a content provider to expose your application data to widgets
- You want to copy and paste complex data or files from your application to other applications
	Content provider allows to control permissions for accessing data.
	
You can use a content provider to abstract away the details for accessing different data sources in your application. For example, your application might store structured records in a SQLite database, as well as video and audio files (we can use FileProvider, it handles file sharing in a secure way, granting temporary permissions to application requesting files). You can use a content provider to access all of this data, if you implement this development pattern in your application.

Content provider provides data in a form like typical SQL relational database. It consists of rows and columns. Rows represent one instance of data that is collect, column represent piece of this data (some field).

So basically, how content provider works, it defines class which act as mediator between app's persistent storage (for example room database) and other application. Content provider receives request and handle it. For example it receives request to pass all the clients. Content provider requests a local database, then transform application specific data to cursor and passes it back.

To access Content Provider you need to use Content Resolver to communicate with the provider as a client.
A common pattern for accessing a ContentProvider from your UI uses a CursorLoader to run an asynchronous query in the background. The Activity or Fragment in your UI calls a CursorLoader to the query, which in turn gets the ContentProvider using the ContentResolver.
![[Pasted image 20230920164229.png]]
In order to acccess provider you usually have to add corresponding permission. You can't request this permission at runtime. Instead, you have to specify that you need this permission in your manifest, using the `[<uses-permission>]` element and the exact permission name defined by the provider.

When you specify this element in your manifest, you are requesting this permission for your application. When users install your application, they implicitly grant this request.

To find the exact name of the read access permission for the provider you're using, as well as the names for other access permissions used by the provider, look in the provider's documentation.

Common example of getting access to content provider

```kotlin
// Queries the UserDictionary and returns results
cursor = contentResolver.query(
        UserDictionary.Words.CONTENT_URI,  // The content URI of the words table
        projection,                        // The columns to return for each row
        selectionClause,                   // Selection criteria
        selectionArgs.toTypedArray(),      // Selection criteria
        sortOrder                          // The sort order for the returned rows
)
```

## Content URIs
A content URI is a URI that identifies data in a provider. Content URIs include the symbolic name of the entire provider—its authority—and a name that points to a table—a path. When you call a client method to access a table in a provider, the content URI for the table is one of the arguments.

In the preceding lines of code, the constant CONTENT_URI contains the content URI of the User Dictionary Provider's Words table. The ContentResolver object parses out the URI's authority and uses it to resolve the provider by comparing the authority to a system table of known providers. The ContentResolver can then dispatch the query arguments to the correct provider.

The ContentProvider uses the path part of the content URI to choose the table to access. A provider usually has a path for each table it exposes.

In the previous lines of code, the full URI for the Words table is:
content://user_dictionary/words
The content:// string is the scheme, which is always present and identifies this as a content URI.
The user_dictionary string is the provider's authority.
The words string is the table's path.

## Building query

The set of columns that the query returns is called a _projection_, and the variable is `mProjection`.

The expression that specifies the rows to retrieve is split into a selection clause and selection arguments. The selection clause is a combination of logical and boolean expressions, column names, and values. The variable is `mSelectionClause`. If you specify the replaceable parameter `?` instead of a value, the query method retrieves the value from the selection arguments array, which is the variable `mSelectionArgs`.
### Display query results

The `[ContentResolver.query()]` client method always returns a `[Cursor]` containing the columns specified by the query's projection for the rows that match the query's selection criteria. A `Cursor` object provides random read access to the rows and columns it contains.

Using `Cursor` methods, you can iterate over the rows in the results, determine the data type of each column, get the data out of a column, and examine other properties of the results.

## Getting data from Cursor

To retrieve data from provider, you iterate over the rows in the Cursor, as shown in the following example:

```kotlin
mCursor?.apply {
    // Determine the column index of the column named "word"
    val index: Int = getColumnIndex(UserDictionary.Words.WORD)

    /*
     * Moves to the next row in the cursor. Before the first movement in the cursor, the
     * "row pointer" is -1, and if you try to retrieve data at that position you get an
     * exception.
     */
    while (moveToNext()) {
        // Gets the value from the column
        newWord = getString(index)

        // Insert code here to process the retrieved word
        ...
        // End of while loop
    }
}
```

## Content provider permissions

A provider's application can specify permissions that other applications must have to access the provider's data. These permissions let the user know what data an application tries to access. Based on the provider's requirements, other applications request the permissions they need in order to access the provider. End users see the requested permissions when they install the application.

```xml
<uses-permission android:name="android.permission.READ_USER_DICTIONARY">
```

Content provider may contain lots of different tables and to construct single object  you may have to perfrom series of operations. For example, to add contact consisting of phone number and name you have to separately create 3 operations:
1) Creating Raw Contact operation
2) Creating name operation
3) Creating phone number operation
```kotlin
	val rawContactValues = ContentValues()
	rawContactValues.putNull(ContactsContract.RawContacts.ACCOUNT_TYPE)
	rawContactValues.putNull(ContactsContract.RawContacts.ACCOUNT_NAME)

	val rawContactUri = contentResolver.insert(ContactsContract.RawContacts.CONTENT_URI, rawContactValues)
	val rawContactId = rawContactUri?.lastPathSegment

	// Insert the display name
	val nameValues = ContentValues()
	nameValues.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId)
	nameValues.put(ContactsContract.Data.MIMETYPE, ContactsContract.CommonDataKinds.StructuredName.CONTENT_ITEM_TYPE)
	nameValues.put(ContactsContract.CommonDataKinds.StructuredName.DISPLAY_NAME, name.text.toString())
	contentResolver.insert(ContactsContract.Data.CONTENT_URI, nameValues)

	// Insert the phone number
	val phoneValues = ContentValues()
	phoneValues.put(ContactsContract.Data.RAW_CONTACT_ID, rawContactId)
	phoneValues.put(ContactsContract.Data.MIMETYPE, ContactsContract.CommonDataKinds.Phone.CONTENT_ITEM_TYPE)
	phoneValues.put(ContactsContract.CommonDataKinds.Phone.NUMBER, phone.text.toString())
	phoneValues.put(ContactsContract.CommonDataKinds.Phone.TYPE, ContactsContract.CommonDataKinds.Phone.TYPE_MOBILE)
	contentResolver.insert(ContactsContract.Data.CONTENT_URI, phoneValues)
```
See the provider documentation with the examples to correctly perform actions you want
## Inserting new data

To insert data into a provider, you call the ContentResolver.insert() method. This method inserts a new row into the provider and returns a content URI for that row. The following snippet shows how to insert a new word into the User Dictionary Provider:

```kotlin
// Defines a new Uri object that receives the result of the insertion
lateinit var newUri: Uri
...
// Defines an object to contain the new values to insert
val newValues = ContentValues().apply {
    /*
     * Sets the values of each column and inserts the word. The arguments to the "put"
     * method are "column name" and "value".
     */
    put(UserDictionary.Words.APP_ID, "example.user")
    put(UserDictionary.Words.LOCALE, "en_US")
    put(UserDictionary.Words.WORD, "insert")
    put(UserDictionary.Words.FREQUENCY, "100")

}

newUri = contentResolver.insert(
        UserDictionary.Words.CONTENT_URI,   // The UserDictionary content URI
        newValues                           // The values to insert
)
```
The columns in this object don't need to have the same data type, and if you don't want to specify a value at all, you can set a column to null using ContentValues.putNull().

## Update data
To update a row, you use a ContentValues object with the updated values, just as you do with an insertion, and selection criteria, as you do with a query. The client method you use is ContentResolver.update(). You only need to add values to the ContentValues object for columns you're updating. If you want to clear the contents of a column, set the value to null.

```kotlin
// Defines an object to contain the updated values
val updateValues = ContentValues().apply {
    /*
     * Sets the updated value and updates the selected words.
     */
    putNull(UserDictionary.Words.LOCALE)
}

// Defines selection criteria for the rows you want to update
val selectionClause: String = UserDictionary.Words.LOCALE + "LIKE ?"
val selectionArgs: Array<String> = arrayOf("en_%")

// Defines a variable to contain the number of updated rows
var rowsUpdated: Int = 0
...
rowsUpdated = contentResolver.update(
        UserDictionary.Words.CONTENT_URI,  // The UserDictionary content URI
        updateValues,                      // The columns to update
        selectionClause,                   // The column to select on
        selectionArgs                      // The value to compare to
)
```

### Delete data

Deleting rows is similar to retrieving row data. You specify selection criteria for the rows you want to delete, and the client method returns the number of deleted rows. The following snippet deletes rows whose app ID matches "user". The method returns the number of deleted rows.

```kotlin
// Defines selection criteria for the rows you want to delete
val selectionClause = "${UserDictionary.Words.APP_ID} LIKE ?"
val selectionArgs: Array<String> = arrayOf("user")

// Defines a variable to contain the number of rows deleted
var rowsDeleted: Int = 0
...
// Deletes the words that match the selection criteria
rowsDeleted = contentResolver.delete(
        UserDictionary.Words.CONTENT_URI,  // The UserDictionary content URI
        selectionClause,                   // The column to select on
        selectionArgs                      // The value to compare to
)
```
## Batch access
Batch access to a provider is useful for inserting a large number of rows, for inserting rows in multiple tables in the same method call, and in general for performing a set of operations across process boundaries as a transaction, called an atomic operation.

To access a provider in batch mode, create an array of ContentProviderOperation objects and then dispatch them to a content provider with ContentResolver.applyBatch(). You pass the content provider's authority to this method, rather than a particular content URI.

This lets each ContentProviderOperation object in the array work against a different table. A call to ContentResolver.applyBatch() returns an array of results.

The description of the ContactsContract.RawContacts contract class includes a code snippet that demonstrates batch insertion.

## Contract classes
A contract class defines constants that help applications work with the content URIs, column names, intent actions, and other features of a content provider. Contract classes aren't included automatically with a provider. The provider's developer has to define them and then make them available to other developers. Many of the providers included with the Android platform have corresponding contract classes in the package android.provider.
```kotlin
val projection : Array<String> = arrayOf(
        UserDictionary.Words._ID,
        UserDictionary.Words.WORD,
        UserDictionary.Words.LOCALE
)
```