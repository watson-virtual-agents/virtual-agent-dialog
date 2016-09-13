# Documentation

## How does Watson Virtual Agent Dialog Communicate with channels (including the Chat Widget) to direct operations
The Client Bot manages communications with a Watson Virtual Agent bot, private variables, and incoming/outgoing message event handling. The Watson Virtual Agent Chat Widget is build on top of the Client Bot SDK and adds a configurable user interface and a set of utilities. Watson Conversation Dialog communicates through the Client BOT SDK to the WVA Chat widget with a series of directives that facilitate the more efficient gathering of user data as well as a mechanism to insulate sensitive information from the IBM Cloud

## IBM Dialog Conversation Directives
###BOT###
- request (call a method to be executed by the bot)
- output.text as a string, as an array
- context variables
- how to represent configurator options (%uid%)
- connect to agent



###UI###
- layout (UI widgets)
- store variables
- input validation (oneOf, typeOf)

###System of Records Access (Client back-end integration)###
- private variables
- action (methods to execute actions like pay bill, update address)

### Client Workspaces
- what do I need to do a client workspace?
- return from Client Workspace to Watson Virtual Agent


##Directives Overview##

### Bot Directives ###

<dl>
<dt><a href="#request">request {}</a></dt>
<dd><p>Request the bot backend to execute a particular action and then call the channel. 'request' is a child of the 'context' object, as explained in detail below. Example: Calculate nearest stores and pass this list to a map widget. The <code> request</code> property of context is used to invoke methods and pass <code>name, args, result</code></p>
</dd>
<dt><a href="#output">output {}</a></dt>
<dd><p>The main output object and its text property to display any text as the output from dialog.</p>
</dd>
<dt><a href="#context">context {}</a></dt>
<dd><p>The context property in the main JSON can be used to store and persist variable values to be used in evaluation of subsequent interactions.</p>
</dd>
<dt><a href="#swapConfigurationProperty">swapConfigurationProperty {}</a></dt>
<dd><p>The request method named swapConfigurationProperty takes the contents of output.text and maps the appropriate configurator ID's value. The syntax used to wrap the property name is the % symbol. For example, when we pass %agent_name% as the node's text, the bot will replace this with the value of agent_name field within the configuration object and the actual value is displayed to the user.</p>
</dd>
</dl>

### Channel Directives and Variables###

<dl>
<dt><a href="#layout">layout {}</a></dt>
<dd><p>Indicates the layout form that should be applied to gather the information.</p>
</dd>
<dt><a href="#inputvalidation">inputvalidation {}</a></dt>
<dd><p>Indicated to the channel that it must validate input to adhere to some rules, currently supported are <code>existIn , oneOf , range</code></p>
</dd>
</dl>

### System of Records Access (Integration) ###

#### Private Variables

Some data items that are necessary to perform some actions or transactions are considered Private Personal Information (PII) (for example last name, SSN, CC number, SSN, etc), and this information needs to remain outside of the IBM Cloud. To facilitate this, Watson Virtual Agent has provided an SDK that should be used to implement channel widgets that can be directed to gather and utilize these data in transcations, but shield it from the IBM DataCenter. From the dialog authoring perspective, there are two key directives to utilize, these are:
<dl>
<dt><a href="#store">store []</a></dt>
<dd><p>This function serves two purposes: <ol>
<li>directs a channel form widget using the label values</li>
<li>directs the widget to store these inputted values on the client server for integration and transactions, WITHOUT returning data to the Watson Virtual Agent Cloud Infrastructure in any way, the dialog waits for a callback with SUCCESS or FAILURE.
</li>
</dd>
<dt><a href="#varname">|&varname| </a></dt>
<dd><p> varname directs the Client Server/Virtual Agent SDK to resolve the variable name from its local 'store' variables,  which it had been previously directoed to store.

You can achieve this by adding '&' in front of a variable names that are used in directives, action or display, but is NOT to be returned to dialog in context.
</li>
</dd>
<dt><a href="#action">action {} </a></dt>
<dd><p>Request the channel to execute a particular action and notify the bot and/or display the results.  Example: Retrieve the values to private variables and populate these in the chat. The <code> action</code> property of output is used to invoke methods and pass <code>name, args</code></p>
</dd>



*Please do NOT pass an PII around in context as this does persist in the IBM Cloud, nor prompt for PII in a dialog of your own construction, please use the following directives to utilize the Virtual Client SDK & server to perform PII & transactions outside the IBM Cloud.*<p>
</dd>
</dl>

### Client Workspaces for Watson Virtual Agent###
<dl>
<dt><a href="#whattodoclient">Requirements</a></dt>
<dd><p>
<ul>
<li>In the initial release, you can only define one client workspace that can be invoked.  This workspace can contain multiple intents.
<li>This workspace must be obtained from the 'IBM Watson Conversation' service.
<li>For any intent that is going to be directed to a client workspace, the client workspace MUST define the exact same intent label in their client workspace.  The client workspace will be invoked with this intent label from Watson Virtual Agent.
<li>Any advanced dialog features (beyond use of 'text' in output) must adhere to the Dialog Directives defined in this document.  Please consult the IBM Watson Virtual Agent - Client SDK and IBM Watson Virtual Agent - UI SDK
<li>upon normal completion a client dialog will return to Watson Virtual Agent control, but at any point control can be returned (see below)

</p>
</dd>

<dt><a href="#returntoMainWorkspace">return from Client Workspace to Watson Virtual Agent </a></dt>
<dd><p> A dialog directive used in <code>context</code> to return immediately to the main workspace from a client workspace at any step in the dialog</p>
</dd>
</dl>

# Directive syntax #

## context
The context property in the main JSON can be used to store and persist variable values. Context is also the place where requests are made to the bot backend to execute a certain method and then call the channel. *Please do NOT pass an PII around in context as this does persist in the IBM Cloud.*
  
### request

The request property of context is used to call a method on the bot backend and then call the channel, passing along any additional information if needed. For example, when dialog invokes a request to retrieve the list of stores near a location, the bot executes this method and then passes along the data for nearest stores for the channel to display.

| Param | Type | Description |
| --- | --- | --- |
| name| <code>String</code> | Name of the method being called |
| args (optional)| <code>JSON Object</code> | List of arguments mapped as name: value |
| result (optional)| <code>String</code> | Result passed back from the bot once the request is executed |

**Example**
```none
Ex: {
	"output": {
		"text": "Select your store",
		"layout": {
			"name": "show-locations"
		}
	},
	"context": {
		"request": {
			"args": {
				"location": "$user_location",
				"location-type": "$location_type"
			},
			"name": "getStoreList"
		}
	}
}
```
In this example, the method _getStoreList_ is called along with the argument values for user location and the type of location. The bot then calculates nearest stores around the location argument's value, and passes along this information to the channel. 

### Persisting variable values
Variable values can be persisted in the context section of the response. Once persisted, context variables can then be used anywhere in the dialog tree in node conditions, as request/action arguments, or can be updated with a new value.

**Example 1 - Creating a variable and assigning a value**  
```none
{
	"output": {},
	"context": {
		"user_location": "<? input.text ?>"
	}
}
```
**Example 2 - Using a context variable as a request/action argument**  
```none
{
	"output": {
		"action": {
			"args": {
				"address_type": "$address_type"
			},
			"name": "updateAddress"
		}
	}
}
```
In the above example, a previously saved context variable named _address_type_'s value is being assigned as an action argument. To do this, we simply use the dialog shorthand for a context variable's value of adding a $ before its name.

### skip_user_input
The construct skip_user_input is used when we do not require a dialog turn to go to the channel. This is useful when we want the bot to execute a request and then come back to dialog instead of going to the UI. 

**Example**  
```none
{
	"output": {},
	"context": {
		"request": {
			"name": "getValue"
		},
		"skip_user_input": true
	}
}
```

<a name="output"></a>
## output
Provides an output dialog from the BOT.


| Param | Type | Description |
| --- | --- | --- |
| text| <code>String</code> | String response from BOT |


**Example**  
```none
{
	"output": {
		"text": "How can I help you?"
	}
}
```

<a name="swapConfigurationProperty"></a>  
## SwapConfigurationProperty
The request method named swapConfigurationProperty takes the contents of output.text and maps the appropriate configurator ID's value.

| Param | Type | Description |
| --- | --- | --- |
| %agent_name% | <code>String</code> | configured agent name |
| %greeting_message% | <code>String</code> | configured greeting message |
| %contact_us% | <code>String</code> | configured message for 'Contact Us' |

**Example**  
```none
{
	"output": {
		"text": "%agent_name%"
	},
	"context": {
		"request": {
			"name": "swapConfigurationProperty"
		}
	}
} 
```

<a name="layout"></a>
## layout
the layout property can be used in the output JSON object to specify the name of the widget to be displayed


| Param | Type | Description |
| --- | --- | --- |
| cc-validator| <code>widget</code> | Credit card widget for Make Payment flow |
| form| <code>widget</code> | Creates a generic form from the field names in the Store object |
| show-locations | <code>widget</code> | Map widget along with store location data for Find Nearest Store flow |
| choose-location-type| <code>widget</code> | UI returns what type of location is being returned - outputs: zipcode, latlong |
| request-geolocation-zipcode| <code>widget</code> | Requests user to input the desired zipcode |
| request-geolocation-latlong| <code>widget</code> | Requests user to share current browser or device location |

**Example 1 - Layout to choose a payment option**  
```none
{
	"output": {
		"text": "How would you like to pay?",
		"layout": {
			"name": "choose"
		},
		"inputvalidation": {
			"oneOf": [
				"Credit Card",
				"Direct Transfer",
				"Paypal"
			]
		}
	}
}
```

**Example 2 - A form layout constructed using Store variables**  
```none
{
	"output": {
		"text": "Please enter your new address",
		"store": [{
			"name": "user_street_address1",
			"label": "Street Address 1"
		}, {
			"name": "user_street_address2",
			"label": "Street Address 2"
		}, {
			"name": "user_locality",
			"label": "Locality/City"
		}, {
			"name": "user_state_or_province",
			"label": "State or Province"
		}, {
			"name": "user_zipcode",
			"label": "Zipcode"
		}],
		"layout": {
			"name": "form"
		}
	}
}
```


<a name="inputvalidation"></a>
## inputvalidation

### Validation using `range`
Validation when you want to signal the channel when the expected input must fall in a certain range of values

| Param | Type | Description |
| --- | --- | --- |
| range| <code>[number,number] | requires input to be in a range of values |

**Example**  
``` none
{
	"output": {
		"text": "OK. That's no problem at all! How much would you like to pay?",
		"inputvalidation": {
			"range": [0.01, 200]
		}
	}
}
```

### Validation using `existIn` or `oneOf`
| Param | Type | Description |
| --- | --- | --- |
| oneOf| <code>[string]</code> | requires input to be one of a range of values |
| existIn| <code>[string]</code> | requires input to be any of a range of values (*for example a multiple selection textbox, select all that apply*)|

**Example - Using oneOf to validate a location layout's response**  
```none
{
	"output": {
		"text": "Before finding a store, please choose how you'd like to provide your location.",
		"layout": {
			"name": "choose-location-type"
		},
		"inputvalidation": {
			"oneOf": [
				"zipcode",
				"latlong"
			]
		}
	}
} 
```


<a name="varname"></a>
##Private Variables and PII

By adding '&' in front of a variable name it indicates that these are Private Variables that are to be collected and used in directives that are not to be returned.

| Param | Type | Description |
| --- | --- | --- |
| &varname | <code>String</code> | Variable used that is not to be returned |

**Example**  
```none
{
	"output": {
		"text": "Hello $fname |&lname|"
	}
}
``` 
where *&lname* is a private variable last name that the widget needs to collect for its purpose, but is not to be placed in context"

<a name="store"></a>
## store
Directs the widget to store these inputted values on the client server for integration and transactions, WITHOUT returning data to the Watson Virtual Agent Cloud Infrastructure in any way, the dialog waits for a callback with SUCCESS or FAILURE.


| Param | Type | Description |
| --- | --- | --- |
| data| <code>name, label</code> | directive to channel to gather and store remotely these values, usually used for performing an external transaction |


**Example**  
```none
{
	"output": {
		"text": "Paying with credit card",
		"store": [{
			"name": "cc_full_name",
			"label": "Name on Card"
		}, {
			"name": "cc_number",
			"label": "Credit Card Number"
		}, {
			"name": "cc_exp_date",
			"label": "Expiry Date"
		}, {
			"name": "cc_code",
			"label": "Security Code"
		}],
		"layout": {
			"name": "cc-validator"
		}
	},
	"context": {
		"payment_method": "creditcard"
	}
}
```
<a name="action"></a>
##action
Using action, we can instruct the channel to execute methods that require access to private variables, system of records, or those that the bot typically cannot handle. For example: completing a payment transaction, updating the billing address, or retrieving private information from the system of records.

| Param | Type | Description |
| --- | --- | --- |
| name| <code>String</code> | Name of the method being called |
| args (optional)| <code>JSON Object</code> | List of arguments mapped as name: value |

**Example 1 - Retrieving private variable values**

```none
{
	"output": {
		"text": "Your account balance is \\$|&bill_amount| due on |&payment_due_date|. What would you like to do?",
		"action": {
			"args": {
				"variables": [
					"bill_amount",
					"payment_due_date"
				]
			},
			"name": "getUserProfileVariables"
		},
		"layout": {
			"name": "choose"
		},
		"variables": "true",
		"inputvalidation": {
			"oneOf": [
				"Pay now",
				"Pay in installments",
				"Defer payment",
				"Autopay"
			]
		}
	}
}
```
In this case, the action `getUserProfileVariables` along with the arguments for which variable values to retrieve, helps populate the text where these variables are tagged as private.


**Example 2 - System of records update using action: Update address**
```none
{
	"output": {
		"action": {
			"args": {
				"address_type": "$address_type"
			},
			"name": "updateAddress"
		}
	}
}
```

**Example 3 - System of records update using action: Pay Bill**

```none
{
	"output": {
		"text": "Processing your payment...",
		"action": {
			"args": {
				"payment_method": "$payment_method"
			},
			"name": "payBill"
		}
	}
}
```
<a name="returntoMainWorkspace"></a>
##Client Workspaces

### return to Watson Virtual Agent from Client Workspace
To return to the Watson Virtual Agent from a client workspace, insert the following into <code>context</code>

| Param | Type | Description |
| --- | --- | --- |
| system.dialog_stack | <code>system.dialog_stack[0] == root</code> | Returns from a client workspace to Watson Virtual Agent |




##Examples##

### Flow included for 'Make a Payment' Intent"

Lets take a look at the dialog flow that is included for the 'Make a Payment' intent.  Here is a view of this flow from the Conversation Tooling

![Image of Make a Payment](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/Payment%20Dialog%20Flow.png)


This is a quite complex flow, that is triggered by an utterance that is determined to be a request to 'Make a Payment', lets look in detail at the first few steps:

![Image of first half](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/Payment%20first-half.png)

You can see immediately that in the first step we do after recognizing the utterance, is give the user their 'current balance' and 'due date' and then prompt them to make a choice on how they would like to pay:

```none
{
	"output": {
		"text": "Your account balance is \\$|&bill_amount| due on |&payment_due_date|. What would you like to do?",
		"action": {
			"args": {
				"variables": [
					"bill_amount",
					"payment_due_date"
				]
			},
			"name": "getUserProfileVariables"
		},
		"layout": {
			"name": "choose"
		},
		"variables": "true",
		"inputvalidation": {
			"oneOf": [
				"Pay now",
				"Pay in installments",
				"Defer payment",
				"Autopay"
			]
		}
	}
}
```

 In the 'text' line, the <code>action</code> directive "getUserProfileVariables" along with the <code>&</code> directive, signals the channel widget that it needs to resolve the variables from its local 'store' named in the args on the client side to present the client.  This data is considered PII and the values of the variables are not to be known to the IBM Virtual Agent system.
 
We also use the <code>layout</code> directive which indicates to the channel widget that it dialog wishes the client to make a choice (as selecting an item from a list) and it must be <code>inputValidation:oneOf</code>.



For this example lets take this case where the user selects: 'Pay Now'

Referring back to the diagram, the users is then prompted on how they would like to pay:
 
```none
{
	"output": {
		"text": "How would you like to pay?",
		"layout": {
			"name": "choose"
		},
		"inputvalidation": {
			"oneOf": [
				"Credit Card",
				"Direct Transfer",
				"Paypal"
			]
		}
	}
}
```


The users previously indicated they wanted to 'Pay Now', so now we again utilize the <code>layout</code> and <code>inputValidation:oneOf</code> to have them choose a payment method.  For our example lets say that the user at this point picks "Credit Card".

We know in designing this dialog that Credit Card will have us collect PII data that we do NOT want to come into the IBM Cloud. For an example, we have created a <code>cc-validator</code> widget which was built on our Virtual Agent SDK and utilizes a Virtual Agent Server component running remotely on the client site.  

We trigger our widget to gather this data that would be necessary to perform an transaction utilizing several directives:

```
{
  "output": {
    "text": "Paying with credit card",
    "store": [
      {
        "name": "cc_full_name",
        "label": "Name on Card"
      },
      {
        "name": "cc_number",
        "label": "Credit Card Number"
      },
      {
        "name": "cc_exp_date",
        "label": "Expiry Date"
      },
      {
        "name": "cc_code",
        "label": "Security Code"
      }
    ],
    "layout": {
      "name": "cc-validator"
    }
  },
  "context": {
    "payment_method": "creditcard"
  }
}
```

So lets examine this, we signal that this request is to be handled by the <code>cc-validator</code> widget, we indicate in <code>context</code> that the widget that the payment_method selected was "creditcard".  Because of PII, we issue the directive <code>store</code> to the widget which indicates to the widget that it must gather this data to perform a potential future transaction, but it is to store it locally and NOT return this to dialog.

In this illustration, you can see that the dialog is simply waiting for a "Success" or "Failure" from this widget, because all the relevent data should be processed on the client side

![confirmation widget](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/credit%20card%20confirmation.png)


The dialog then leads them through whether they would like a receipt, and how that would be delivered. At this point in the dialog we know that the client side indicated that it has gathered all the data needed and they are in the client <code>store</code>, and we have dialoged confirmation from the end user, so now we utilize <code>action</code> to call the client side implemenation for the payment_method to actually perform the transaction:
.

```
{
  "output": {
    "text": "Processing your payment...",
    "action": {
      "args": {
        "payment_method": "$payment_method"
      },
      "name": "payBill"
    }
  }
}
```





