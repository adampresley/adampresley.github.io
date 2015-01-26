---
layout: post
title: Exporting Blog Entries as Markdown
date: 2014-05-06 05:25:00
author: Adam Presley
status: Published
tags: development python
slug: exporting-blog-entries-as-markdown
---
Since I write my blog posts in Markdown format I decided I wanted a way to backup my posts in a way that is human readable. The result is a feature in Texo to export my blog entries as zipped up Markdown files.

![Screenshot of Export Interface](/assets/adampresley/images/posts/texo-export-markdown-1.png)

<!-- excerpt -->

When I click on the button in the above screenshot all my blog entries are saved as Markdown files, zipped up, and presented to me as a download. The final zip is organized into folders of **YEAR**/**MONTH**.

![Export Zip File](/assets/adampresley/images/posts/text-export-markdown-2.png)

The code to do this is pretty simple. First I have a function that constructs the Markdown given a post dictionary.

{% highlight python %}
def generateMarkdownFile(post):
   result = """Title: %s
Date: %s
Author: %s
Status: %s
Tags: %s
Slug: %s

%s""" % (
         post["title"],
         post["publishedDateTime"],
         post["author"],
         post["status"],
         post["tagList"],
         post["slug"],
         post["content"],
      )

   filename = os.path.join(config.UPLOAD_PATH, post["slug"]) + ".md"

   with open(filename, "w") as blogFile:
      blogFile.write(result)

   return filename
{% endhighlight %}

This function writes out a file to my temporary upload path with the post data converted to a format that Texo actually knows how to import. It is simple and human readable. The next part was to write a controller action to get my posts, write Markdown files, then zip them up and serve. This method is a bit big and needs a bit of refactoring, but it does the job for now.

{% highlight python %}
@route("/admin/utilities/exportmarkdownfiles", method="GET")
@route("/admin/utilities/exportmarkdownfiles", method="POST")
@view("admin-export-markdown-files.html")
@requireSession
def adminExportMarkdownFiles():
   logger = logging.getLogger(__name__)

   if "btnExport" in request.all:
      posts = postservice.getAllPosts()
      zipfilePath = os.path.join(config.UPLOAD_PATH, "blog-posts.zip")

      zf = zipfile.ZipFile(zipfilePath, mode="w")
      filenames = []

      try:
         #
         # Write each post to a file, adding each file to a ZIP
         #
         for post in posts:
            filename = postservice.generateMarkdownFile(post=post)
            filenames.append(filename)

            writtenFilename = "%s/%s/%s" % (post["publishedYear"], post["publishedMonth"], os.path.basename(filename),)
            zf.write(filename, compress_type=compression, arcname=writtenFilename)

      except Exception as e:
         logger.error("There was an error writing zipfile: %s", e.message)

      finally:
         zf.close()

      #
      # Clean out markdown files
      #
      for filename in filenames:
         try:
            os.remove(filename)
         except Exception as e:
            logger.error("Unable to remove %s" % (filename,))

      #
      # Serve up the ZIP file as a download
      #
      return static_file(os.path.basename(zipfilePath), root=config.UPLOAD_PATH)

   return {
      "title": "Export Markdown Files"
   }
{% endhighlight %}

Basically this method starts up a zipfile output, gets all posts, and creates Markdown files for each post, adding to the zip file after each Markdown file is created. At the end I clean out all the generated Markdown files and serve the ZIP file back as a static download.

Also, as a final note, I added a fun easter-egg to my site. Try out the old Konami code on the home page. :) Cheers!