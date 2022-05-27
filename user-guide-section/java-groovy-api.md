# Java/Groovy API

File Field is compatible with all Jira automation apps: [ScriptRunner](https://marketplace.atlassian.com/apps/6820/scriptrunner-for-jira), [Automation for Jira](https://marketplace.atlassian.com/apps/1215460/automation-for-jira-server), [JSU](https://marketplace.atlassian.com/apps/5048/jsu-automation-suite-for-jira-workflows), etc.\
You can use the FileManager java component to manage your files programmatically from java/groovy scripts.

## Plain Java Example <a href="#java-groovyapi-plainjavaexample" id="java-groovyapi-plainjavaexample"></a>

```java
import com.apwide.file.api.FileManager;
...
import com.atlassian.jira.component.*;
 
 
// Get FileManager instance
FileManager fileManager = ComponentAccessor.getComponent(FileManager.class);
 
// Binary content to upload
InputStream inputStream = new ByteArrayInputStream("any content".getBytes(StandardCharsets.UTF_8));
// Name of the file to upload
String fileName = "Readme.txt";
// Size limit of file to upload (ex: 10Mo)
long maxFileSize = 10*1000*1000;
 
// UPLOAD
String fileId = fileManager.upload(fileName, inputStream, maxFileSize);
 
// DONWLOAD
InputStream downloadedFile = fileManager.download(fileId);
 
// GET PATH
String filePath = fileManager.getPath(fileId);
 
// CHECK IF IT EXISTS
boolean exists = fileManager.exists(fileId);
 
// DELETE
boolean deleted = fileManager.delete(fileId);234567891011121314151617181920212223242526272829
```



## Script Runner Groovy Example  <a href="#java-groovyapi-scriptrunnergroovyexample" id="java-groovyapi-scriptrunnergroovyexample"></a>

This example script shows how to create/upload 2 files and add them to an existing File Field customfield.

N.B. Adding multiple uploaded files into 1 single customfield is supported since **version 3.1.0**.

You should always set the value of a File customfield using a unique String:

* containing 1 file id.\
  Example: "1568037617780\_10000\_w3c\_home.gif"
* or containing multiple file ids separated by a comma.\
  Example: "1568037617780\_10000\_w3c\_home.gif,1568037613455\_10000\_another\_file.gif"

```java
import com.onresolve.scriptrunner.runner.customisers.WithPlugin
import com.onresolve.scriptrunner.runner.customisers.PluginModule
 
@WithPlugin("com.apwide.document.file-field")
 
import com.apwide.file.api.FileManager;
import com.atlassian.jira.component.*;
import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
 
 
@PluginModule
FileManager fileManager

// Size limit of file to upload (ex: 10Mo)
long maxFileSize = 10*1000*1000;
 
// create and upload first file
String textFileContent = "My first example file"
InputStream inputStream = new ByteArrayInputStream(textFileContent.getBytes(StandardCharsets.UTF_8))
String fileId_1 = fileManager.upload("My First example file.txt", inputStream, maxFileSize)
 
// create and upload second file
String textFileContent = "My second example file"
InputStream inputStream = new ByteArrayInputStream(textFileContent.getBytes(StandardCharsets.UTF_8))
String fileId_2 = fileManager.upload("My Second example file.txt", inputStream, maxFileSize)
 
 
 
// Update the customfield
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
 
 
def issueManager = ComponentAccessor.getIssueManager()
def issue = issueManager.getIssueObject("HSP-31")
def customFieldManager = ComponentAccessor.getCustomFieldManager()
def fileField = customFieldManager.getCustomFieldObjects(issue).find {it.name == "file"}
 
// get current value of the file field for an Issue if you need it (file ids. String with following format: "fileId1,fileId2")
def currentFileIds = fileField.getValue(issue)
 
if (fileField) {
    def changeHolder = new DefaultIssueChangeHolder()
    // update customfield's value using a String containing file ids separated by a comma
    fileField.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(fileField), fileId_1 + "," + fileId_2), changeHolder)
}
```

## Copy an Attachment to a File Field custom field

This ScriptRunner groovy script shows how to copy an existing attachment to a File Field customfield:

```groovy
import com.onresolve.scriptrunner.runner.customisers.WithPlugin
import com.onresolve.scriptrunner.runner.customisers.PluginModule

@WithPlugin("com.apwide.document.file-field")

import com.apwide.file.api.FileManager;
import com.atlassian.jira.component.*;
import com.atlassian.jira.issue.attachment.Attachment
import com.atlassian.jira.issue.attachment.FileSystemAttachmentDirectoryAccessor
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder


def attachmentManager = ComponentAccessor.getAttachmentManager()
def attachmentDirectoryAccessor = ComponentAccessor.getComponent(FileSystemAttachmentDirectoryAccessor.class)

@PluginModule
FileManager fileManager

// Load the issue you want to update
def issueManager = ComponentAccessor.getIssueManager()
Issue issue = issueManager.getIssueObject("GOT-1")

// find attachment by file name
String fileName = "Existing Attachment Name.txt"
List<Attachment> lastAttachments = attachmentManager.getAttachments(issue)
Attachment attachment = lastAttachments.find({ attachment -> attachment.getFilename().equals(fileName) })
log.warn "Found Attachment: ${attachment}"
if (attachment){
    // get attachment binary file
    log.warn "Files found in issue folder: ${attachmentDirectoryAccessor.getAttachmentDirectory(issue).listFiles()}"
    def file = attachmentDirectoryAccessor.getAttachmentDirectory(issue)
            .listFiles()
            .find({ it-> it.getName().equals(""+attachment.id)})
    log.warn "File found: ${file}"
    if (file){
        // Size limit of file to upload (ex: 10Mo)
        long maxFileSize = 10*1000*1000;
        // upload the attachment binary file to File Field and get its unique id
        String fileId = fileManager.upload(fileName, new FileInputStream(file), maxFileSize)
        log.warn "FileId of newly updated file: ${fileId}"

        // find the target File Field to update
        def customFieldManager = ComponentAccessor.getCustomFieldManager()
        def fileField = customFieldManager.getCustomFieldObjects(issue).find {it.name == "Existing File Field customfield name"}
        log.warn "File Field found: ${fileField}"

        if (fileField) {
            def changeHolder = new DefaultIssueChangeHolder()
            // update customfield's value using a String containing the file id
            fileField.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(fileField), fileId), changeHolder)
            log.warn "File field successfully updated with the new fileId"
        }
    }
}
```

## Copy File Fields to the Jira attachments field <a href="#copy-file-fields-to-jira-attachments-field" id="copy-file-fields-to-jira-attachments-field"></a>

Ram Kumar Aravindakshan (Adaptavist) answered this question on the Jira Community:

[How to copy the contents of an Apwide File Field to Jira's general attachment field](https://community.atlassian.com/t5/Marketplace-Apps-Integrations/How-to-copy-the-contents-of-an-Apwide-File-Field-to-Jira-s/qaq-p/1726659)

For your requirement, you could try using a ScriptRunner Listener.

The IssueCreate event / IssueUpdated event can be used to copy the attachment from the Apwide File Field to the Attachment field either during the creation of the Issue or when the Issue is being updated.

Below is a print screen of the Listener configuration:

![](https://community.atlassian.com/t5/image/serverpage/image-id/148050i0FD8563F12ADE7C9/image-size/large?v=v2\&px=999)

Below is a working sample code for your reference:

```groovy
import com.atlassian.jira.issue.attachment.Attachment
import com.atlassian.renderer.util.FileTypeUtil
import com.onresolve.scriptrunner.runner.customisers.WithPlugin
import com.onresolve.scriptrunner.runner.customisers.PluginModule

@WithPlugin("com.apwide.document.file-field")

import com.apwide.file.api.FileManager;
import com.atlassian.jira.component.*;
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.issue.attachment.CreateAttachmentParamsBean
import com.apwide.document.file.FileDescriptor
import org.apache.commons.io.FilenameUtils

@PluginModule
FileManager fileManager

def issue = event.issue as MutableIssue
log.warn "Current Issue: ${issue}"

def final APWIDE_FILE_FIELD_NAME = "Resume" // set you own Apwide File Field name here

def customFieldManager = ComponentAccessor.customFieldManager
def attachmentManager = ComponentAccessor.attachmentManager
def loggedInUser = ComponentAccessor.jiraAuthenticationContext.loggedInUser

def customField = customFieldManager.getCustomFieldObjectsByName(APWIDE_FILE_FIELD_NAME)[0]
log.warn "Apwide customField: ${customField}"

def customFieldValue = customField.getValue(issue).toString().split(",") as List<String>
log.warn "Apwide customField value: ${customFieldValue}"

if (customFieldValue.isEmpty())
    return

for (fileId in customFieldValue){
    log.warn "Current file id: ${fileId}"

    List<Attachment> existingAttachments = attachmentManager.getAttachments(issue)
    log.warn("Existing attachments: ${existingAttachments}(number of attachments: ${existingAttachments.size()})")

    if (fileManager.exists(fileId)){
        def filePath = fileManager.getPath(fileId)
        log.warn "File path: ${filePath}"

        def fileDescriptor = new FileDescriptor(fileId)
        log.warn "File descriptor: ${fileDescriptor}"
        log.warn "File name: ${fileDescriptor.fileName}"
        log.warn "File type: ${fileDescriptor.fileType}"
        log.warn "File extension: ${fileDescriptor.fileExtension}"
        log.warn "Thumbnailable: ${fileDescriptor.thumbnailable}"
        log.warn "User id: ${fileDescriptor.userId}"
        log.warn "Timestamp: ${fileDescriptor.timestamp}"

        def sourceFile = new File(filePath)
        def attachmentParams = new CreateAttachmentParamsBean(sourceFile,fileDescriptor.fileName,null,loggedInUser,issue,null,fileDescriptor.thumbnailable,null,new Date(fileDescriptor.timestamp),true)

        Attachment existingAttachmentWithSameName = existingAttachments.find({Attachment it ->  it.getFilename() == fileDescriptor.fileName} )
        log.warn "Existing attachment with same name: ${existingAttachmentWithSameName}"
        if (existingAttachmentWithSameName != null){
            log.warn "Already existing attachment with same name"
            if (existingAttachmentWithSameName.getCreated().getTime() == fileDescriptor.timestamp){
                log.warn "Exact same file already exists, do nothing"
            }
            else{
                log.warn "Delete the existing file with the same name"
                attachmentManager.deleteAttachment(existingAttachmentWithSameName)
                log.warn "Attach the new file with the same name"
                attachmentManager.createAttachment(attachmentParams)
            }
        }
        else{
            attachmentManager.createAttachment(attachmentParams)
            log.warn "File successfully added to attachments!"
        }
    }
}


```

Below are some test print screens:

* When the Apwide attachment is added, it is automatically added to Jira's attachment, as well as shown in the image below:

![](https://community.atlassian.com/t5/image/serverpage/image-id/148051iF2C27B4F3F70FAEF/image-size/large?v=v2\&px=999)

* Although the attachment displays as the Actual file name added when it is stored in the Folder, it will display a timestamp as shown in the image below (which is why the Jira attachment appears to be so):

![](https://community.atlassian.com/t5/image/serverpage/image-id/148052i50856210309806FE/image-size/large?v=v2\&px=999)

* However, in the Jira attachments folder, it displays as a regular attachment file as shown below:

![](https://community.atlassian.com/t5/image/serverpage/image-id/148053iB769021FF7C2624E/image-size/large?v=v2\&px=999)

## JavaDoc <a href="#java-groovyapi-javadoc" id="java-groovyapi-javadoc"></a>

### String upload(String fileName, InputStream file, long maxFileSize)

Upload file

* **Parameters:**
  * `fileName` — name of the file
  * `file` — file content
  * `maxFileSize` — max file size allowed (in bytes)
* **Returns:** generated unique id of the uploaded file

### boolean exists(String fileId)

Check if file exists

* **Parameters:** `fileId` — unique id of the file
* **Returns:** true if file exists, false otherwise

### InputStream download(String fileId) throws FileNotFoundException

Donwload file

* **Parameters:** `fileId` — unique id of the file
* **Returns:** content of the file
* **Exceptions:** `FileNotFoundException` — if file could not be found

### boolean delete(String fileId) throws FileNotFoundException

Delete file

* **Parameters:** `fileId` — unique id of the file
* **Returns:** true if file was deleted
* **Exceptions:** `FileNotFoundException` — if file could not be found

### String getPath(String fileId) throws FileNotFoundException

Get file path from file id

* **Parameters:** `fileId` — unique id of the file
* **Returns:** file path of the file
* **Exceptions:** `FileNotFoundException` — if file could not be found



## Java API Interface

```java
package com.apwide.file.api;
 
import java.io.FileNotFoundException;
import java.io.InputStream;
 
public interface FileManager {
    /**
     * Upload file
     * @param fileName name of the file
     * @param file file content
     * @param maxFileSize max file size allowed (in bytes)
     * @return generated unique id of the uploaded file
     */
    String upload(String fileName, InputStream file, long maxFileSize);
 
    /**
     * Check if file exists
     * @param fileId unique id of the file
     * @return true if file exists, false otherwise
     */
    boolean exists(String fileId);
 
    /**
     * Donwload file
     * @param fileId unique id of the file
     * @return content of the file
     * @throws FileNotFoundException if file could not be found
     */
    InputStream download(String fileId) throws FileNotFoundException;
 
    /**
     * Delete file
     * @param fileId unique id of the file
     * @return true if file was deleted
     * @throws FileNotFoundException if file could not be found
     */
    boolean delete(String fileId) throws FileNotFoundException;
 
    /**
     * Get file path from file id
     * @param fileId unique id of the file
     * @return file path of the file
     * @throws FileNotFoundException if file could not be found
     */
    String getPath(String fileId) throws FileNotFoundException;
}
```
