assembla2jira
=============

`assembla2jira` will convert an Assembla project export into a JIRA-importable JSON file.  The imported data is all tickets, comments, and files attached to tickets.  Primary ticket status is preservered (eg: open or closed), as is entry date/time, reporter, asignee, and commenter.  Ticket state transitions (eg: from "New" to "Accepted, in progress") are not kept.

## assembla2jira syntax

```
$ assembla2jira -i <path to Assembla ticket export directory> \
                -u <path to Assembla user ID to JIRA user map (JSON)> \
                -p <path to new JIRA project definition (JSON) \
                -a<URL "prefix" for exported attachment files> > jira-compatable.json
```
## Requirements

1. PHP available at the command line.
2. Access to a web server you can throw up the attachments from Assembla on so JIRA can download them.
3. An Assembla account and backup file
4. A JIRA account to create a new project in.

## Steps

- Export from Assembla: goto the Assembla project > "Admin" tab > "Backup and Data Export" 
- Click "Export my data"
- Reload the page in 5-15 minutes, the download *could* be ready.  Or it might take longer.
- When it's ready download it and untar/gzip it (ex: `tar -zxf <filename>.tar.gz`) someplace convienent.
- Upload to the web server mentioned above in step 2 so JIRA can download the attachment data: `scp -Cr <directory created by the untar/gzip> <your server>:<path to where the files will hang out>`
- Set the permissions on the remote files just in case; `ssh <your server> 'chmod -R a+rX <path to where the files will hang out>'`
- Copy [projectPrototype.json](projectPrototype.json) to a new file that you'll edit to create the JIRA project definition, it's relatively undocumented but be sure to fill in at least `name`, `key`, and `description`.  `name` is the friendly human name of hte project, `key` is the short abbreviation that's used to reference the project internally.  It's *very important* to be sure the `name` and `key` are unique for the project you're importing otherwise this may overwrite existing data.
- Create the [userMap.json](userMap.json) which will map Assembla IDs to JIRA users, the Assembla ID will be something like "bAnUJ2ORKr4Az9acwqjQWU" while the JIRA "name" is the username and something like `bsr` and "fullname" is the full name and something like `Brent Rieck`. Unfortunately all of your Assembla user IDs will need to be filled in and finding all of them in Assembla is difficult.  If it comes down to a few IDs you can't find you can assign multiple IDs to the same JIRA user or to a JIRA user which doesn't exist. 
- Run `assembla2jira`:
```
$ assembla2jira -i <path to Assembla ticket export directory> \
                -u userMap.json \
                -p <path to new JIRA project definition (JSON) \
                -a"http://substancedev:substancedev@assemblaexport.substancedev.com" > jira-compatable.json
```
- Import the generated JSON into JIRA to create project: https://<your JIRA url>/secure/admin/views/JsonSetupPage!default.jspa?externalSystem=com.atlassian.jira.plugins.jira-importers-plugin:jsonImporter
- There *may* be import errors, JIRA will list them and you can see them via the "download a detailed log" link shown after an import.