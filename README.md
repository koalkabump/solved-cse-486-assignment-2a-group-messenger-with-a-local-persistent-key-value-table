Download Link: https://assignmentchef.com/product/solved-cse-486-assignment-2a-group-messenger-with-a-local-persistent-key-value-table
<br>
The teaching staff hopes you had fun working on PA1! If you got frustrated, we feel for you and believe us, we were there too. While it is expected to be frustrated in the beginning, we promise you, it will get better and you will enjoy more and more as you do it. You might even start enjoying reading the Android documentation because it *is* actually the single best place to get great information about Android. We do hope, though, that you now understand a bit more about what it means to write networked apps on Android.

Now back to the assignment: this assignment builds on the previous simple messenger and points to the next assignment. You will design a group messenger that can send message to multiple AVDs and store them in a permanent key-value storage. ​Once again, please follow everything exactly. Otherwise, it might result in getting no point for this assignment.

The rest of the description can be long. Please don’t “tl;dr”! Please read to the end first and get the overall picture. Then please revisit as you go!

<h1>Step 0: Importing the project template</h1>

Unlike the previous assignment, we will have strict requirements for the UI as well as a few other components. In order to provide you more help in meeting these requirements, we have a project template you can import to Android Studio.

<ol>

 <li>Download <u>​</u><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/GroupMessenger1.zip">the project template zip file</a></u><u>​</u> to a temporary directory.</li>

 <li>Extract the template zip file and copy it to your Android Studio projects directory.</li>

 <li>Make sure that you copy the correct directory. After unzipping, the directory name should be “GroupMessenger1”, and right underneath, it should contain a number of directories and files such as “app”, “build”, “gradle”, “build.gradle”, “gradlew”, etc.</li>

 <li>After copying, delete the downloaded template zip file and unzipped directories and files. This is to make sure that you do not submit the template you just downloaded. (There were many people who did this before.)</li>

 <li>Open the project template you just copied in Android Studio.</li>

 <li>Use the project template for implementing all the components for this assignment.</li>

</ol>

<h1>Step 1: Writing a Content Provider</h1>

Your first task is to write a content provider. This provider should be used to store all messages, but the abstraction it provides should be a general key-value table. Before you start, please read the following to understand the basics of a content provider: <u><a href="https://developer.android.com/guide/topics/providers/content-providers.html">http://developer.android.com/guide/topics/providers/content-providers.html</a></u>




Typically, a content provider supports basic SQL statements. However, you do not need to do it for this course. You will use a content provider as a table storage that stores (key, value) pairs.




The following are the requirements for your provider:

<ol>

 <li>You should not set any permission to access your provider.​ This is very important since if you set a permission to access your content provider, then our testing program cannot test your app. The current template takes care of this; so as long as you do not change the template, you will be fine.</li>

 <li>Your provider’s URI should be</li>

</ol>

“content://edu.buffalo.cse.cse486586.groupmessenger1.provider”, which means that any app should be able to access your provider using that URI. To simplify your implementation, your provider does not need to match/support any other URI pattern. This is already declared in the project template’s AndroidManifest.xml.

<ol start="3">

 <li>Your provider should have two columns.

  <ol>

   <li>The first column should be named as “key” (an all lowercase string without the quotation marks). This column is used to store all keys.</li>

   <li>The second column should be named as “value” (an all lowercase string without the quotation marks). This column is used to store all values.</li>

   <li>All keys and values that your provider stores should use the string data type.</li>

  </ol></li>

 <li>Your provider should only implement insert() and query(). All other operations are not necessary.</li>

 <li>Since the column names are “key” and “value”, any app should be able to insert a &lt;key, value&gt; pair as in the following example:</li>

</ol>




<table width="0">

 <tbody>

  <tr>

   <td width="624">ContentValues keyValueToInsert = new ContentValues(); // inserting &lt;”key-to-insert”, “value-to-insert”&gt; keyValueToInsert.put(“key”, “key-to-insert”); keyValueToInsert.put(“value”, “value-to-insert”); Uri newUri = getContentResolver().insert(providerUri,    // assume we already created a Uri object with our provider URI     keyValueToInsert);</td>

  </tr>

 </tbody>

</table>

<strong><em> </em></strong>

<ol start="6">

 <li>If there’s a new value inserted using an existing key, you need to keep ​only the most</li>

</ol>

recent value​. You should not preserve the history of values under the same key.

<ol start="7">

 <li>Similarly, any app should be able to read a &lt;key, value&gt; pair from your provider with query(). Since your provider is a simple &lt;key, value&gt; table, we are not going to follow the Android convention; your provider should be able to answer queries as in the following example:</li>

</ol>




<table width="0">

 <tbody>

  <tr>

   <td width="624">Cursor resultCursor = getContentResolver().query(providerUri,    // assume we already created a Uri object with our provider URInull,                // no need to support the ​<em>projection </em>​parameter“key-to-read”,    // we provide the key directly as the ​<em>selection</em>​ parameter     null,                // no need to support the ​<em>selectionArgs</em>​ parameter     null                 // no need to support the ​<em>sortOrder </em>​parameter);</td>

  </tr>

 </tbody>

</table>




Thus, your query() implementation should read the ​<em>selection</em>​ parameter and use it as the key to retrieve from your table. In turn, the Cursor returned by your query() implementation should include only one row with two columns using your provider’s column names, i.e., “key” and “value”. You probably want to use android.database.MatrixCursor instead of implementing your own Cursor.

<ol start="8">

 <li>Your provider should store the &lt;key, value&gt; pairs using one of the data storage options. The details of possible data storage options are in</li>

</ol>

<u><a href="https://developer.android.com/guide/topics/data/data-storage.html">http://developer.android.com/guide/topics/data/data-storage.html</a></u><u>​</u><a href="https://developer.android.com/guide/topics/data/data-storage.html">.</a> You can choose any option; however, the easiest way to do this is probably use the internal storage with the key as the file name and the value stored in the file.

<ol start="9">

 <li>After implementing your provider, you can verify whether or not you are meeting the requirements by clicking “PTest” provided in the template. You can take a look at OnPTestClickListener.java to see what tests it does.</li>

 <li>If your provider does not pass PTest, there will be no point for this portion of the assignment.</li>

</ol>

<h1>Step 2: Implementing Multicast</h1>

The final step is implementing multicast, i.e., sending messages to multiple AVDs.​ ​The requirements are the following.

<ol>

 <li>Your app should multicast every user-entered message to all app instances (including​ the one that is sending the message​). ​In the rest of the description, “multicast” always means sending a message to all app instances.</li>

 <li>Your app should be able to send/receive multiple times.</li>

 <li>Your app should be able to handle concurrent messages.</li>

 <li>As with PA1, we have fixed the ports &amp; sockets.

  <ol start="10000">

   <li>Your app should open one server socket that listens on 10000.</li>

   <li>You need to use run_avd.py and set_redir.py to set up the testing environment.

    <ol>

     <li>python run_avd.py 5</li>

     <li>python set_redir.py 10000</li>

    </ol></li>

   <li>The grading will use 5 AVDs. The redirection ports are 11108, 11112, 11116, 11120, and 11124.</li>

   <li>You should just hard-code the above 5 ports and use them to set up connections.</li>

   <li>Please use the code snippet provided in PA1 on how to determine your local AVD.

    <ol>

     <li>emulator-5554: “5554” ii. emulator-5556: “5556” iii. emulator-5558: “5558” iv. emulator-5560: “5560” v. emulator-5562: “5562”</li>

    </ol></li>

   <li>Your app needs to assign a sequence number to each message it receives. The sequence number should start from 0 and increase by 1 for each message.</li>

   <li>Each message and its sequence number should be stored as a &lt;key, value&gt; pair in your content provider. The key should be the sequence number for the message (as a string); the value should be the actual message (again, as a string).</li>

   <li>All app instances should store every message and its sequence number individually.</li>

   <li>For your debugging purposes, you can display all the messages on the screen. However, there is no grading component for this.</li>

   <li>Please read the notes at the end of this document. You might run into certain problems, and the notes might give you some ideas about a couple of potential problems.</li>

  </ol></li>

</ol>

<h1>Testing</h1>




We have testing programs to help you see how your code does with our grading criteria. If you find any rough edge with the testing programs, please report it on Piazza so the teaching staff can fix it. The instructions are the following:

<ol>

 <li>Download a testing program for your platform. If your platform does not run it, please report it on Piazza.

  <ol start="8">

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/groupmessenger1-grading.exe">Windows</a></u>​: We’ve tested it on 32- and 64-bit Windows 8.</li>

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/groupmessenger1-grading.linux">Linux</a></u>​: We’ve tested it on 32- and 64-bit Ubuntu 12.04.</li>

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/groupmessenger1-grading.osx">OS X</a></u>:​ We’ve tested it on 32- and 64-bit OS X 10.9 Mavericks.</li>

  </ol></li>

 <li>Before you run the program, please make sure that you are running five AVDs. ​python py 5​ will do it.</li>

 <li>Please also make sure that you have installed your GroupMessenger1 on all the AVDs.</li>

 <li>Run the testing program from the command line.</li>

 <li>On your terminal, it will give you your partial and final score, and in some cases, problems that the testing program finds.</li>

 <li>You might run into a debugging problem if you’re reinstalling your app from Android Studio. This is because your content provider will still retain previous values even after reinstalling. This won’t be a problem if you uninstall explicitly before reinstalling; uninstalling will delete your content provider storage as well. In order to do this, you can uninstall with this command:</li>

</ol>




$ ​adb uninstall ​edu.buffalo.cse.cse486586.groupmessenger1