# MailDB-Data-Analysis
## Analyzing an EMAIL Archive and vizualizing the data using the D3 JavaScript library.

I used the mail archive stored at http://mbox.dr-chuck.net/ for this project.

The first step is to spider the repository. The base URL is hard-coded in the *gmane.py* and is hard-coded to the Sakai developer list. The *gmane.py* file operates as a spider in that it runs slowly and retrieves one mail message per second so as to avoid getting throttled by the website. It stores all of its data in a database and can be interrupted and re-started as often as needed.

*Note: Windows has difficulty in displaying UTF-8 characters in the console so for each console window you open, you may need to type the following command before running this code:*

`chcp 65001`

The *content.sqlite data* is pretty raw, with an innefficient data model, and not compressed. This is intentional as it allows you to look at *content.sqlite* to debug the process.

The second process is running the program *gmodel.py*.  *gmodel.py* reads the rough/raw data from *content.sqlite* and produces a cleaned-up and well-modeled version of the 
data in the file *index.sqlite*.  The file index.sqlite will be much smaller (often 10X smaller) than *content.sqlite* because it also compresses the header and body text.

Each time *gmodel.py* runs - it completely wipes out and re-builds *index.sqlite*, allowing you to adjust its parameters and edit the mapping tables in *content.sqlite* to tweak the data cleaning process.

The *gmodel.py* program does a number of data cleaning steps:

- Domain names are truncated to two levels for .com, .org, .edu, and .net other domain names are truncated to three levels. So si.umich.edu becomes umich.edu and caret.cam.ac.uk becomes cam.ac.uk. Also mail addresses are forced to lower case and some of the @gmane.org address like the following:

   `arwhyte-63aXycvo3TyHXe+LvDLADg@public.gmane.org`
   
   are converted to the real address whenever there is a matching real email address elsewhere in the message corpus.

- If you look in the *content.sqlite* database there are two tables that allow you to map both domain names and individual email addresses that change over the lifetime of the email list.  For example, Steve Githens used the following email addresses over the life of the Sakai developer list:

  - s-githens@northwestern.edu
  - sgithens@cam.ac.uk
  - swgithen@mtu.edu

  We can add two entries to the Mapping table

  - s-githens@northwestern.edu ->  swgithen@mtu.edu
  - sgithens@cam.ac.uk -> swgithen@mtu.edu

And so all the mail messages will be collected under one sender even if they used several email addresses over the lifetime of the mailing list.

You can also make similar entries in the DNSMapping table if there are multiple DNS names you want mapped to a single DNS. In the Sakai data I add the following
mapping:

`iupui.edu -> indiana.edu`

So all the folks from the various Indiana University campuses are tracked together.

You can re-run the *gmodel.py* over and over as you look at the data, and add mappings to make the data cleaner and cleaner. When you are done, you will have a nicely indexed version of the email in *index.sqlite*. This is the file to use to do data analysis. With this file, data analysis will be really quick.

The first, simplest data analysis is done using *gbasic.py*. It basically finds top Email list participants and top Email organizations.

Problematic data in *index.sqlite* can be fixed by updating the Mapping table and DNSMapping table in *content.sqlite* and re-running *gmodel.py*.

*gword.py* is used to get frequency data and convert it to a JavaScript file, *gword.js* which you can visualize using the file *gword.htm*.

A second visualization is in *gline.py*. It visualizes email participation by organizations over time. Its output is written to *gline.js* which is visualized using *gline.htm*.
