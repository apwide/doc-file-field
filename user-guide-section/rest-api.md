# REST API

## Download an existing file <a href="#restapi-downloadanexistingfile" id="restapi-downloadanexistingfile"></a>

You must use this enpdoint and make a **GET**:

```css
GET {server:port}/{your-jira-context}/rest/apwide/document/1.0/file/{issueId}/{fieldId}/{fileId}
```

* **issueId**: numerical id of the issue (ex: 10234)
* **fieldId**: key of the apwide custom field (ex: customfield\_10345)
* **fileId** (since version 3.1.+): id of the file to download (ex: 1544184648025\_10000\_file-field-1.0.5.jar.zip)

N.B. you must be authenticated and your user must have permissions to view the issue. Check general documentation of JIRA API below if you are not familiar with that.

![](<../.gitbook/assets/image (34).png>)

## Upload a new file <a href="#restapi-uploadanewfile" id="restapi-uploadanewfile"></a>

You must make a POST:

```css
POST {server:port}/{your-jira-context}/rest/apwide/document/1.0/file?fileName=YourFileName.example 
```

* **Body**: content of your file (text or binary)
* **fileName**: the desired file name that will be displayed in JIRA

It will then return you the _fileId_. The _fileId_ is the unique key of the document. It is also the value that is stored in JIRA as custom field value. You must keep this _fileId_ to update your issues in the next step.

![](<../.gitbook/assets/image (35).png>)

## Update your File Field Customfield with previously uploaded file(s) <a href="#restapi-updateyourapwidefieldcustomfieldwithpreviouslyuploadedfile-s" id="restapi-updateyourapwidefieldcustomfieldwithpreviouslyuploadedfile-s"></a>

You must make a PUT:

```css
PUT {server:port}/{your-jira-context}/rest/api/2/issue/{issueIdOrKey}
```

Example of body (single file) :

```javascript
{
  "fields" : {
    "customfield_10602": "your generated file id"
  }
}
```

Example of body (multiple files, since **version 3.1.+**) :

```css
{
  "fields" : {
    "customfield_10602": "generated_file_id_1,generated_file_id_2,generated_file_id_3"
  }
}
```

![](<../.gitbook/assets/image (36).png>)

## Download an existing file by its unique ID <a href="#restapi-downloadanexistingfilebyitsuniqueid" id="restapi-downloadanexistingfilebyitsuniqueid"></a>

You must use this enpdoint and make a **GET**:

```css
GET {server:port}/{your-jira-context}/rest/apwide/document/1.0/file/{documentId}
```

* **documentId**: unique id of the file stored in the customfield (ex: 1544184648025\_10000\_file-field-1.0.5.jar.zip)

**N.B. You must be authenticated and your user must have Jira administration permissions**. Check general documentation of JIRA API below if you are not familiar with that.

![](<../.gitbook/assets/image (38).png>)

## Remove an existing file by its unique ID <a href="#restapi-removeanexistingfilebyitsuniqueid" id="restapi-removeanexistingfilebyitsuniqueid"></a>

You must use this enpdoint and make a **DELETE**:

```css
DELETE {server:port}/{your-jira-context}/rest/apwide/document/1.0/file/{documentId}
```

* **documentId**: unique id of the file stored in the customfield (ex: 1544184648025\_10000\_file-field-1.0.5.jar.zip)

**N.B. You must be authenticated and your user must have Jira administration permissions**. Check general documentation of JIRA API below if you are not familiar with that.

![](<../.gitbook/assets/image (39).png>)

## Jira Documentation & Tools <a href="#restapi-jiradocumentationandtools" id="restapi-jiradocumentationandtools"></a>

General documentation of JIRA API:\
[https://docs.atlassian.com/software/jira/docs/api/REST/7.6.1/](https://docs.atlassian.com/software/jira/docs/api/REST/7.6.1/)

Check also this free add-on that is very convenient to browse the REST API of JIRA: [https://marketplace.atlassian.com/plugins/com.atlassian.labs.rest-api-browser/server/overview](https://marketplace.atlassian.com/plugins/com.atlassian.labs.rest-api-browser/server/overview)

**Note that you have to disable "Show only public API's" in order to see API's of third party plugins like Apwide File Fields.**

Screenshots above were done with the REST API Browser plugin.
