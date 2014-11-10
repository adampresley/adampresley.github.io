---
layout: post
title: Images in SQL
date: 2009-02-09 11:21:00
author: Adam Presley
status: Published
tags: development sql c#
slug: images-in-sql
---

So I was presented with a task today of extracting images that were
stored in a Microsoft SQL Server 2000 database and save them as JPEG
files. Here is a bit of code that will do this. In this code sample I am
assuming domain authenticated SQL login. The table name here will be
called **Photos** and the column names **id**, **type**, and **photo**.  
  
    :::c#
    SqlConnection connection;
    SqlCommand command;
    SqlDataReader reader;
     
    string outputDirectory = "C:\\TestLocation";
    PictureBox pic = new PictureBox();
     
    using (connection = new SqlConnection("data source=MyServer;initial catalog=Database;persist security info=True;packet size=4096;Trusted_Connection=True")) {
        connection.Open();
     
        command = new SqlCommand("SELECT id, type, photo FROM Photos", connection);
        command.CommandType = CommandType.Text;
        command.CommandTimeout = 3000;
         
        reader = command.ExecuteReader();
     
        while (reader.Read()) {
           try {
              byte[] rawBytes = (byte[]) reader["photo"];
              MemoryStream memStream = new MemoryStream();
              memStream.Write(rawBytes, 0, rawBytes.Length);
         
              string filename = outputDirectory + "\\image-" + reader["id"].ToString() + "-" + reader["type"].ToString() + ".jpg";
         
              pic.Image = Image.FromStream(memStream);
              pic.Image.Save(filename, System.Drawing.Imaging.ImageFormat.Jpeg);
         
              memStream.Close();
           }
           catch (Exception ex) {
           }
        }
    }
  
This C# code will loop over a record set and save the images stored in
the table as JPEG files. Happy coding!