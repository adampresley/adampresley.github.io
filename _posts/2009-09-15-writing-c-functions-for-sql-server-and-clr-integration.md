---
layout: post
title: Writing C# Functions for SQL Server and CLR Integration
date: 2009-09-15 10:46:00
author: Adam Presley
status: Published
tags: development sql c#
slug: writing-c-functions-for-sql-server-and-clr-integration
---

An interested problem was presented to me today where a database table
in SQL Server 2005 (now moved to 2008) had a trigger attached to a table
that would encrypt a credit card number on insert or update. The
encryption routine used the sp_OACreate stored procedure to make use of
an older Microsoft cryptography library called CAPICOM. With the move to
SQL Server 2008, however, we also upgraded our OS to 64-bit Windows. It
turns out that this CAPICOM component does not have support for the
64-bit world, and started to cause a world of hurt, causing credit card
numbers to not go into the database. This, or course, is bad.

So I now decided that to tackle this problem I would make use of SQL
Server's ability to access .NET assemblies and their methods. The first
order of business to get this started is to enable CLR integration in
your SQL Server instance. By default this is disabled, so two simple
statements will do the trick to turn it on.

	:::sql
	sp_configure 'clr enabled', 1;
	GO
	reconfigure
	GO

Now that you have CLR integration turned on let's get to the fun stuff.
Code. Fire up your C# environment ladies! Now, if you are like me, and
you don't have the super cool Enterprise edition of Visual Studio, you
will have to make due without all the little helper wizards that handle
deployment of your code to a SQL Server instance, and you will have to
do it manually. Don't worry, it isn't hard.

First, create a new Class Library project. This will create a project
that contains a file named "Class.cs". I normally rename that to
something nicer, like "Crypto.cs". C# will also create a namespace
definition for your new project by default. In our case we won't be
needing that, so remove it (don't forget your open and close curly
braces).

From here we will need a few references for our project. Those are:
System, System.Data.SqlTypes, System.Security.Cryptography, System.Text,
System.IO, and Microsoft.SqlServer.Server. That will look like this.

	:::c#
	using System;
	using System.IO;
	using System.Text;
	using System.Data.SqlTypes;
	using Microsoft.SqlServer.Server;
	using System.Security.Cryptography;

SQL Server expects a class defined as **partial**, and a series of
methods that are **public static**, and have the attribute
**[SqlFunction()]** for creating a .NET managed function for SQL Server.
Functions are easy to implement because they simply return simple
values, unlike stored procedures (which is beyond the scope of this
post).

For this example I will be creating two functions: Encrypt and Decrypt.
Each function that we wish to expose will contain the SqlFunction
attribute as described above, will be public, and also be marked as
static. Here are what they look like.

	:::c#
	[SqlFunction()]
	public static string Encrypt(string Input, string Password, string Salt) {
		if (Input == null || Input.Length <= 0) return "";

		Rfc2898DeriveBytes keyGen = __createKeyGen(Password, Salt);
		ICryptoTransform transformer = __createEncryptor(keyGen);
		byte[] transformed = __transform(Encoding.Default.GetBytes(Input), transformer);

		return Convert.ToBase64String(transformed);
	}

	[SqlFunction()]
	public static string Decrypt(string Input, string Password, string Salt) {
		if (Input == null || Input.Length <= 0) return "";
		Rfc2898DeriveBytes keyGen = __createKeyGen(Password, Salt);
		ICryptoTransform transformer = __createDecryptor(keyGen);
		byte[] transformed = __transform(Convert.FromBase64String(Input), transformer);

		return Encoding.Default.GetString(transformed);
	}

Clearly these methods are higher level and depend on other methods to do
the actual dirty work, but they are the methods that will be exposed to
SQL Server, and will serve to describe what is happening.

Each method takes three arguments. The first is an input string. For the
**Encrypt** method this will be the plain text value that needs to be
encrypted. For the **Decrypt** method this will be the base64 encoded
encrypted string that needs to be decrypted. The *Password* argument is
the password that will be used to generate the encryption and decrypt
values, followed by the *Salt* value used along with the *Password* to
generate the encryption key.

As seen in the code above there are four primary methods used to perform
these actions: **__createKeyGen**, **__createEncryptor**,
**__createDecryptor**, and **__transform**. Let's look at each.

	:::c#
	private static Rfc2898DeriveBytes __createKeyGen(string Password, string Salt) {
		return new Rfc2898DeriveBytes(Password, Encoding.Default.GetBytes(Salt));
	}

The **__createKeyGen** method takes a password and salt values to
create a key generator object based on the RFC 2989 specification. This
will be used by the encryptor and decryptor to generate the key and
initialization vector (IV).

	:::c#
	private static ICryptoTransform __createEncryptor(Rfc2898DeriveBytes KeyGen) {
		TripleDES provider = TripleDES.Create();
		return provider.CreateEncryptor(KeyGen.GetBytes(16), KeyGen.GetBytes(16));
	}

The **__createEncryptor** method creates an object used to encrypt
values based on the encryption provider, which in this case is
TripleDES. Notice how this method expects a Rfc2989DeriveBytes object
which we generate from the **__createKeyGen**, and that we use it to
generate a 128-bit (16 bytes) key and IV.

The same applies to the **__createDecryptor** method, except that we
create an object that decrypts instead of encrypts.

	:::c#
	private static ICryptoTransform __createDecryptor(Rfc2898DeriveBytes KeyGen) {
		TripleDES provider = TripleDES.Create();
		return provider.CreateDecryptor(KeyGen.GetBytes(16), KeyGen.GetBytes(16));
	}

Now comes the fun part. Time to perform encryption or decryption.

	:::c#
	private static byte[] __transform(byte[] Input, ICryptoTransform Transformer) {
		MemoryStream ms = new MemoryStream();
		byte[] result;

		CryptoStream writer = new CryptoStream(ms, Transformer, CryptoStreamMode.Write);
		writer.Write(Input, 0, Input.Length);
		writer.FlushFinalBlock();

		ms.Position = 0;
		result = ms.ToArray();

		ms.Close();
		writer.Close();
		return result;
	}

This method takes a byte array for the input, and an encryptor or
decryptor (thus the ICryptoTransform interface argument). Right out of
the box we create a MemoryStream object. This is where the final
encrypted or decrypted result will be stored. We also create a byte
array called *result* that we will get an array of bytes from the
MemoryStream into to send back as the result.

From here we create an instance of the CryptoStream class and we will
call it *writer*. The first parameter specifies that the MemoryStream
*ms* will be where the CryptoStream writer will put its data. The second
parameter is the encryptor or decryptor object. The third parameter
specifies that we wish to write to the MemoryStream.

From here we use the **Write** method, passing it the input byte array,
start offset, and byte array length, to write the bytes to the
CryptoStream writer to be encrypted or decrypted.

After this we reset our MemoryStream position, and read the data from it
into a byte array, which will be the resulting encrypted or decrypted
value. Close up shop to clean up, and return the results.

It is worth noting that the **Encrypt** method converts the encrypted
byte array to a Base64 string for portability, and the **Decrypt**
method expects a Base64 encoded string to decrypt.

Now we get to plug it into SQL Server. For your database there is a
branch in the Object Explorer named "Programmability". Under here there
is a branch named "Assemblies". After you compile your code you will
need to register your assembly in SQL Server. To do this, right click on
the Assemblies folder and select "New Assembly...". The resulting dialog
will have a Path option where you can browse for your compiled DLL file.
Find this, then click "Ok". If all is well you will see your DLL name
under the "Assemblies" folder.

Now you need to create an actual SQL Server function wrapper for the CLR
function. For the sake of this bit of code let's assume the name of the
assembly is "MyCrypto", and the name of the class is "Crypto". This can
be done like so.

	:::sql
	CREATE FUNCTION Encrypt(@Input nvarchar(255), @Password nvarchar(255), @Salt nvarchar(255)) RETURNS nvarchar(255)
	EXTERNAL NAME MyCrypto.Crypto.Encrypt;

	CREATE FUNCTION Encrypt(@Input nvarchar(255), @Password nvarchar(255), @Salt nvarchar(255)) RETURNS nvarchar(255)
	EXTERNAL NAME MyCrypto.Crypto.Decrypt;

This bit of SQL will create SQL Server functions that reference the .NET
assembly. To use this function would look something like this.

	:::sql
	SELECT dbo.Encrypt('some important value', 'myPassword', 'salt is nice') AS encryptedValue;

And there you have it. A cool, powerful feature, and not hard to do!
Happy coding!

	:::c#
	using System;
	using System.IO;
	using System.Text;
	using System.Data.SqlTypes;
	using Microsoft.SqlServer.Server;
	using System.Security.Cryptography;

	public partial class Crypto {
		[SqlFunction()]
		public static string Encrypt(string Input, string Password, string Salt) {
			if (Input == null || Input.Length <= 0) return "";

			Rfc2898DeriveBytes keyGen = __createKeyGen(Password, Salt);
			ICryptoTransform transformer = __createEncryptor(keyGen);
			byte[] transformed = __transform(Encoding.Default.GetBytes(Input), transformer);

			return Convert.ToBase64String(transformed);
		}

		[SqlFunction()]
		public static string Decrypt(string Input, string Password, string Salt) {
			if (Input == null || Input.Length <= 0) return "";

			Rfc2898DeriveBytes keyGen = __createKeyGen(Password, Salt);
			ICryptoTransform transformer = __createDecryptor(keyGen);
			byte[] transformed = __transform(Convert.FromBase64String(Input), transformer);

			return Encoding.Default.GetString(transformed);
		}

		[SqlFunction()]
		public static string Hash(string Input) {
			if (Input == null || Input.Length <= 0) return "";

			StringBuilder result = new StringBuilder();
			SHA1 provider = SHA1.Create();
			byte[] __result = provider.ComputeHash(Encoding.Default.GetBytes(Input));

			foreach (Byte b in __result)
				result.Append(String.Format("{0:x2}", b));

			return result.ToString();
		}

		private static Rfc2898DeriveBytes __createKeyGen(string Password, string Salt) {
			return new Rfc2898DeriveBytes(Password, Encoding.Default.GetBytes(Salt));
		}

		private static ICryptoTransform __createEncryptor(Rfc2898DeriveBytes KeyGen) {
			TripleDES provider = TripleDES.Create();
			return provider.CreateEncryptor(KeyGen.GetBytes(16), KeyGen.GetBytes(16));
		}

		private static ICryptoTransform __createDecryptor(Rfc2898DeriveBytes KeyGen) {
			TripleDES provider = TripleDES.Create();
			return provider.CreateDecryptor(KeyGen.GetBytes(16), KeyGen.GetBytes(16));
		}

		private static byte[] __transform(byte[] Input, ICryptoTransform Transformer) {
			MemoryStream ms = new MemoryStream();
			byte[] result;

			CryptoStream writer = new CryptoStream(ms, Transformer, CryptoStreamMode.Write);
			writer.Write(Input, 0, Input.Length);
			writer.FlushFinalBlock();

			ms.Position = 0;
			result = ms.ToArray();

			ms.Close();
			writer.Close();
			return result;
		}
	}
