
# Disposable Email Domain List

In this repository you will find a collection of temporary email domains. You can import them into your newsletter software and filter from which addresses new subscribers subscribe.

The aim of the project is to allow newsletter owners to easily filter / block junk email addresses (like '10 minute mail').


## Files

In this repository, you will find two important files:

- [domains.txt](domains.txt) - list in TXT format
- [domains.json](domains.json) - list in JSON format

## VBA Script to use this with Microsoft Outlook
1. Open Microsoft Outlook.
2. Press Alt + F11 on your keyboard. This will open the Visual Basic for Applications (VBA) Editor.
3. Create a New Module: In the VBA Editor, click on "Insert" in the top menu >> Choose "Module." This will insert a new module into the project.
4. Delete whatever is autopolulated and insert the below code.
5. Save the VBA Project: Click on the "Save" button in the VBA Editor or press Ctrl + S.
6. Close the VBA Editor by clicking the "X" button or pressing Alt + Q.

```
Option Explicit

Private WithEvents inboxItems As Outlook.Items

Private Sub Application_Startup()
    Dim ns As Outlook.NameSpace
    Set ns = Application.GetNamespace("MAPI")
    
    Dim inbox As Outlook.Folder
    Set inbox = ns.GetDefaultFolder(olFolderInbox)
    Set inboxItems = inbox.Items
End Sub

Private Sub inboxItems_ItemAdd(ByVal item As Object)
    If TypeOf item Is MailItem Then
        FilterSpamEmails item
    End If
End Sub

Sub FilterSpamEmails(item As MailItem)

    Dim spamFolder As Outlook.MAPIFolder
    Dim mail As Outlook.MailItem
    Dim headerLines As Variant
    Dim line As Variant
    Dim domains As Collection
    Dim domain As Variant
    
    ' Create a new folder under Inbox named "Spoofing Emails"
    ' If the folder already exists, the existing one will be used
    On Error Resume Next
    Set spamFolder = Application.Session.GetDefaultFolder(olFolderInbox).Folders("Spoofing Emails")
    If spamFolder Is Nothing Then
        Set spamFolder = Application.Session.GetDefaultFolder(olFolderInbox).Folders.Add("Spoofing Emails")
    End If
    On Error GoTo 0
    
    ' Load spam domains from the web
    Set domains = GetDomainsFromWeb("https://raw.githubusercontent.com/unkn0w/disposable-email-domain-list/main/domains.txt")
    
    ' Check the headers of the incoming mail
    Set mail = item
    headerLines = Split(mail.PropertyAccessor.GetProperty("http://schemas.microsoft.com/mapi/proptag/0x007D001E"), vbCrLf)
    
    ' Loop through each line in the headers
    For Each line In headerLines
        ' Check if the line contains a known spam domain
        For Each domain In domains
            If InStr(line, domain) > 0 Then
                mail.Move spamFolder
                Exit Sub
            End If
        Next domain
    Next line
    
    ' Clean up
    Set spamFolder = Nothing
    Set mail = Nothing

End Sub

Private Function GetDomainsFromWeb(url As String) As Collection
    Dim xmlhttp As Object
    Set xmlhttp = CreateObject("MSXML2.XMLHTTP")
    
    xmlhttp.Open "GET", url, False
    xmlhttp.send
    
    Dim stream As Object
    Set stream = CreateObject("ADODB.Stream")
    
    stream.Open
    stream.Type = 1 ' adTypeBinary
    stream.Write xmlhttp.responseBody
    stream.Position = 0
    stream.Type = 2 ' adTypeText
    stream.Charset = "utf-8"
    
    Dim domains As Collection
    Set domains = New Collection
    
    Dim lines As Variant
    lines = Split(stream.ReadText, vbCrLf)
    
    Dim line As Variant
    For Each line In lines
        domains.Add Trim(line)
    Next line
    
    stream.Close
    
    Set GetDomainsFromWeb = domains
End Function
```

## Contributing

Contributions are always welcome!

Try to add new domains to `domains.txt` file in small chunks (up to 10 domains at once).

**Please always provide proof why such domain is 'spammy'!** (maybe some link to temp-mail service that offers this domain?)

Please don't sort domains in alphabetic order. Just put your update at the end of the list.

## FAQ

#### Can I add email forwarders to this list?

No. In this list I would like to keep only temporary email addresses, like this one that expires after 10 minutes or are one-time-use.

Forwarders (like anonaddy.com, icloud.com or relay.firefox.com) are just fine.

#### How to remove domain from the list?

Just send pull request with description, why domain should not be on the list

## Authors

- [@unknow](https://www.github.com/unkn0w)


