---
layout: post
title: Rekindling My Love Affair with C++, part 1
date: 2012-04-11 14:51:00
author: Adam Presley
status: Published
tags: development c++
slug: rekindling-my-love-affair-with-c-part-1
---
Yesterday I was reading various articles online and stumbled across
[0x10c.com](http://0x10c.com/). Although I won't go into any details of
that project (you'll just have to click to see) it did make me slightly
nostalgic for one of the first languages I learned, C/C++. I happen to be
one of those types that when trying a new language or product I look for a
small project to get my feet wet. Only this time I wanted to reacquaint myself
with an old friend. So I decided I would take my MailSlurper project,
written in Java, and convert it to C++. This is clearly not the simplest
thing I could have picked but it is familiar and sounds fun. Last night
I began the conversion.

For step one I decided to address the configuration file that allows the user to specify the host,
port and other information used to configure how the MailSlurper
application runs. Immediately it occurs to me that I have forgotten much
of the **detail** behind how to do some things in C++. For example I had
forgotten the syntax for defining a class in a header file and the
implementation in a CPP file. The end result looks like this:

**IniParser.h**

{% highlight cpp %}
#ifndef __INI_PARSER_H__

#define __INI_PARSER_H__ 1

#include <stdio.h>
#include <string.h>
#include <iostream>
#include <iomanip>
#include <map>
#include <fstream>
#include <string>

using namespace std;

class IniParser {
	private:
		char* __filename;
		map<string, string> __kvp;
		ifstream __fp;

		void __openFile(char* filename);
		void __closeFile();
		char* __readLine();

	public:
		IniParser(const char* filename);
		~IniParser();

		void parseFile(char* filename);
		string getValue(string key);
};

#endif
{% endhighlight %}

**IniParser.cpp**

{% highlight cpp %}
#include "IniParser.h"

void IniParser::__openFile(char* filename) {
	__fp.open(filename);

	if (!__fp) {
		cerr << "Can't open file.";
	}
}

void IniParser::__closeFile() {
	__fp.close();
}

char* IniParser::__readLine() {
	char* line = new char[255];
	__fp >> line;
	return line;
}

IniParser::IniParser(const char* filename) {
	__filename = new char[512];
	strcpy(__filename, filename);
	parseFile(__filename);
}

IniParser::~IniParser() {
	delete __filename;
}

void IniParser::parseFile(char* filename) {
	char* line;
	string key, value;

	__kvp.clear();
	__openFile(filename);

	while (strlen(line = __readLine())) {
		key = string(strtok(line, "="));
		value = string(strtok(NULL, "="));

		printf("key = %s, value = %s\n", key.c_str(), value.c_str());
		__kvp[key] = value;
	}

	__closeFile();
}

string IniParser::getValue(string key) {
	return __kvp[key];
}
{% endhighlight %}

**Main.cpp**

{% highlight cpp %}
#include <string>
#include "IniParser/IniParser.h"

using namespace std;

int main(void) {
	IniParser* parser = new IniParser("properties.ini");
	string host, port;

	host = parser->getValue("host");
	port = parser->getValue("port");
	printf("%s @ %s\n", host.c_str(), port.c_str());

	delete parser;
}
{% endhighlight %}

There are two things I am reminded of after doing this part:

1.  How much functionality and boilerplate stuff higher level languages
    do for you
1.  How much control you **don't** have in higher languages

Stay tuned for the next part!
