---
layout: post
title: Advanced Data Validation Using ExtJs
date: 2008-05-07 19:34:00
author: Adam Presley
status: Published
tags: development javascript extjs
slug: advanced-data-validation-using-extjs
---
While working on my personal project tonight I finally got around to
going over some of the new examples in the [ExtJs](http://www.extjs.com) 
[documentation](http://extjs.com/deploy/dev/docs/). In this case they are 
demonstrating having a form with a password field, and a confirmation 
password field, and how you can validate that the second password entered 
is the same as the first. That's fine and dandy, but I did need it to 
do a little more. I need this validation to verify that the entered password 
is at least five characters, and contains at least one number or special 
character from a specific set.  
  
So here is an excerpt of the form definitions. I have the first password
box, and a second for confirmation. Notice the "vtype" is set to
"password". This is key.  
  
{% highlight javascript %}
{
	fieldLabel: 'Password',
	id: 'password',
	name: 'password',
	allowBlank: false,
	xtype: 'textfield',
	inputType: 'password',
	vtype: 'password',
	width: 200,
	maxLength: 64
},
{
	fieldLabel: 'Confirm',
	id: 'passwordConfirm',
	name: 'passwordConfirm',
	xtype: 'textfield',
	inputType: 'password',
	vtype: 'password',
	allowBlank: false,
	width: 200,
	maxLength: 64,
	initialPasswordField: 'password'
}
{% endhighlight %}

Now, prior to this you need to define the behavior for the password
validation. This is done by vtypes. In ExtJs a vtype is basically an
object with custom data validation functions that return true or false
upon success of validation. They also include a message to display in a
QuickTip when a data validation error occurs. Here is the code I used to
validate the passwords according to the rules mentioned above.  
  
{% highlight javascript %}
Ext.apply(Ext.form.VTypes, {
	password: function(value, field) {
		if (field.initialPasswordField) {
			var pwd = Ext.getCmp(field.initialPasswordField);
			this.passwordText = 'Confirmation does not match your intial password entry.';
			return (value == pwd.getValue());
		}

		this.passwordText = 'Passwords must be at least 5 characters, containing either a number, or a valid special character (!@#$%^&*()-_=+)';
		var hasSpecial = value.match(/[0-9!@#\$%\^&\*\(\)\-_=\+]+/i);
		var hasLength = (value.length >= 5);
		return (hasSpecial && hasLength);
	},
	passwordText: 'Passwords must be at least 5 characters, containing either a number, or a valid special character (!@#$%^&*()-_=+)'
});
{% endhighlight %}

If you aren't familiar with Ext.apply this is a function that takes two
objects and combines them. And here is no exception. We are combining a
new object with **OUR** definition of the password validation with the
existing VTypes object.  
  
If you noticed the property "initialPasswordField" in the confirmation
field, you'll see that the first thing our password validation function
does is check to see if the form field has this property. If it does
that means we are validating data for the confirmation field, and we
want to make sure it matches the first password box.  
  
If the current field being validated is not the confirmation field, but
instead the initial password field, I am defining two variables:
*hasSpecial*, and *hasLength*. These are the two criteria I am testing
against. The *hasLength* is obvious; true or false if the value is
greater than or equal to five characters in length. The *hasSpecial* is
simply a regular expression that checks to see if the value has at least
one number, or one character in the list: *!@\#\$%\^&\*()-\_=+*.  
  
Also notice the *passwordText* property. This is the default message to
use when the data validation fails. You will also see that I am changing
the password text based on which field you are working with when the
data validation occurs.  
  
I, for one, am quite excited about the power and flexibility of how data
validation and vtypes works with ExtJs. Cheers, and happy coding!
