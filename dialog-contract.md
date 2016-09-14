# Watson Virtual Agent Dialog Design #

The Watson Virtual Agent (WVA) dialog works in concert with the Watson Virtual Agent (WVA) chat widget, which was built on top of the Watson Virtual Agent (WVA) Client SDK. The WVA dialog communicates via JSON, extending the base IBM Watson Conversation dialog to issue a series of directives that are interpreted by the chat widget to facilitate the more efficient gathering of user data as well as a mechanism to insulate personally identifiable information (PII) from the IBM Cloud.

## IBM Virtual Agent Conversation Dialog Directives
###Bot###
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

### Custom dialogs
- what do I need to build a custom dialog?
- what to return from custom dialog to Watson Virtual Agent


##Directives Overview##

### Bot Directives ###

<dl>
<dt><a href="#request">request {}</a></dt>
<dd><p>Request the bot back end to execute a particular action and then call the channel. 'request' is a child of the 'context' object, as explained in detail below. Example: Calculate nearest stores and pass this list to a map widget. The <code> request</code> property of context is used to invoke methods and pass <code>name, args, result</code></p>
</dd>
<dt><a href="#output">output {}</a></dt>
<dd><p>The main output object and its text property to display any text as the output from dialog.</p>
</dd>
<dt><a href="#context">context {}</a></dt>
<dd><p>The context property in the main JSON can be used to store and persist variable values to be used in evaluation of subsequent interactions.</p>
</dd>
<dt><a href="#swapConfigurationProperty">swapConfigurationProperty {}</a></dt>
<dd><p>The request method named swapConfigurationProperty takes the contents of output.text and maps the appropriate configurator ID's value. The syntax used to wrap the property name is the % symbol. For example, when we pass %agent_name% as the node's text, the bot will replace this with the value of the agent_name field within the configuration object, and the actual value is displayed to the user.</p>
</dd>
</dl>

### Channel Directives and Variables###

<dl>
<dt><a href="#layout">layout {}</a></dt>
<dd><p>Indicates the layout form that should be applied to gather the information.</p>
</dd>
<dt><a href="#inputvalidation">inputvalidation {}</a></dt>
<dd><p>Indicates to the channel that it must validate input to adhere to some rules. Currently supported rules are <code>existIn , oneOf , range</code></p>
</dd>
</dl>

### System of Records Access (Integration) ###

#### Private Variables

Some data items that are necessary to perform some actions or transactions are considered private personal information (PII) (examples include last name, Social Security number, and credit card number). This information needs to remain outside the IBM Cloud. To facilitate this, Watson Virtual Agent has provided an SDK that should be used to implement channel widgets that can be directed to gather and utilize these data in transcations, but shield it from the IBM Cloud. From the dialog authoring perspective, there are three key directives:
<dl>
<dt><a href="#store">store []</a></dt>
<dd><p>This function serves two purposes: <ol>
<li>directs a channel form widget using the label values</li>
<li>directs the widget to store the input values on the client system for integration and transactions, without returning data to the Watson Virtual Agent cloud infrastructure in any way. The dialog waits for a callback with SUCCESS or FAILURE.
</li>
</dd>
<dt><a href="#varname">|&varname| </a></dt>
<dd><p> varname directs the client system to resolve the variable name from its local 'store' variables, replacing it with the value is was previously directed to store.

You can achieve this by adding '&' in front of a variable name that is used in a directive, action, or display, but is not to be returned in the dialog context.
</li>
</dd>
<dt><a href="#action">action {} </a></dt>
<dd><p>Request the channel to execute a particular action, notify the bot, and/or display the results (for example, retrieve the values of private variables and populate these in the chat). The <code> action</code> property of output is used to invoke methods and pass <code>name, args</code></p>
</dd>



*Please do NOT pass any PII within the dialog context, or prompt for PII within a dialog of your own construction, as this would result in PII persisting in the IBM Cloud. Instead, please use the provided directives to utilize the Virtual Client SDK & server to perform PII & transactions outside the IBM Cloud.*<p>
</dd>
</dl>

### Client Workspaces for Watson Virtual Agent###
#####Requirements
<dd><p>
<ul>
<li>In the initial release, you can only define one custom dialog to invoke.  This dialog can contain multiple intents.
<li>The workspace containing the dialog must be obtained from the IBM Watson Conversation service.
<li>For any intents handled by a custom dialog, the dialog must specify the same intent names that appear on the intent configuration page.  The Watson Virtual Agent will invoke the dialog using these intent names.
<li>Any advanced dialog features (beyond use of 'text' in output) must adhere to the dialog directives defined in this document.  Please consult the IBM Watson Virtual Agent - Client SDK and IBM Watson Virtual Agent - UI SDK for more information.
<li>Upon normal completion, a custom dialog will return to Watson Virtual Agent control, but at any point control can be returned (see below).

</p>
</dd>
#####Return to Watson Virtual Agent from Client Workspace
<dd><p> A dialog directive used in <code>context</code> to return immediately to the IBM dialog from a custom dialog</p>
</dd>
</dl>

# Directive details#

## context
The <code>context</code> property in the main JSON can be used to store and persist variable values. Context is also the place where requests are made to the bot back end to execute a certain method and then call the channel. 

*Do not pass PII within the dialog context, as this does persist in the IBM Cloud.*
  
### request

The request property of context is used to call a method on the bot back end and then call the channel, passing along any additional information if needed. For example, when the dialog invokes a request to retrieve the list of stores near a location, the bot executes this method and then passes along the data for nearest stores for the channel to display.

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
In the above example, a previously saved context variable named _address_type_'s value is being assigned as an action argument. To do this, we simply use the dialog shorthand for a context variable's value: adding a $ before its name.

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
Provides an output dialog from the bot.


| Param | Type | Description |
| --- | --- | --- |
| text| <code>String</code> | String response from bot |


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

Adding '&' in front of a variable name indicates that it is a private variable that is to be collected and used in directives but not returned.

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
where *&lname* is a private variable that the widget needs to collect for its purposes, but is not to be placed in context"

<a name="store"></a>
## store
Directs the widget to store these values on the client system for integration and transactions, without returning data to the Watson Virtual Agent Cloud infrastructure in any way. The dialog waits for a callback with SUCCESS or FAILURE.


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
Using action, we can instruct the channel to execute methods that require access to private variables, systems of record, or those that the bot typically cannot handle (for example, completing a payment transaction, updating the billing address, or retrieving private information from a system of record).

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
In this case, the action `getUserProfileVariables` along with the arguments for variable values to retrieve, helps populate the text where these variables are tagged as private.


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
To return to the Watson Virtual Agent IBM dialog from a custom dialog, insert the following into <code>context</code>

| Param | Type | Description |
| --- | --- | --- |
| system.dialog_stack | <code>system.dialog_stack[0] == root</code> | Returns from a custom dialog to Watson Virtual Agent |




##Examples##

### Flow included for 'Make a Payment' Intent"

Let's take a look at the dialog flow that is included for the 'Make a Payment' intent.  Here is a view of this flow from the Conversation service tooling.

![Image of Make a Payment](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/Payment%20Dialog%20Flow.png)


This is a complex flow that is triggered by an utterance that is determined to be a request to make a payment. Let's look in detail at the first few steps:

![Image of first half](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/Payment%20first-half.png)

You can see immediately that in the first step after recognizing the utterance, the dialog gives the user their current balance and due date, and then prompts them to make a choice for how they would like to pay:

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

 In the 'text' line, the <code>action</code> directive "getUserProfileVariables", along with the <code>&</code> directive, signals the channel widget that it needs to resolve the variables from its local 'store' named in the args on the client side to present the client.  This data is considered PII, and the values of the variables are not to be returned to the IBM Virtual Agent system.
 
We also use the <code>layout</code> directive, which indicates to the channel widget that the dialog should prompt the user to make a choice (such as selecting an item from a list), and that the choice must be <code>inputValidation:oneOf</code>.



For this example, let's take the case where the user selects: 'Pay Now'

Referring back to the diagram, the user is then prompted to choose how they would like to pay:
 
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


The user previously indicated they wanted to 'Pay Now', so now we again use the <code>layout</code> and <code>inputValidation:oneOf</code> to have them choose a payment method.  For our example, let's say that the user at this point picks "Credit Card".

We know in designing this dialog that the Credit Card choice will have us collect PII data that we do not want to come into the IBM Cloud. For an example, we have created a <code>cc-validator</code> widget that was built on our Virtual Agent SDK and utilizes a Virtual Agent Server component running remotely on the client site.  

We trigger our widget to gather the data that is necessary to perform a transaction utilizing several directives:

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

So let's examine this. We signal that this request is to be handled by the <code>cc-validator</code> widget, and we indicate in <code>context</code> that the payment_method selected was "creditcard".  Because of PII, we issue the directive <code>store</code> to the widget, which indicates to the widget that it must gather this data to perform a potential future transaction, but it is to store it locally and not return it to the dialog.

In this illustration, you can see that the dialog is simply waiting for a "Success" or "Failure" from this widget, because all the relevant data is processed on the client side.

![confirmation widget](https://github.com/watson-virtual-agents/virtual-agent-dialog/blob/master/doc-images/credit%20card%20confirmation.png)


The dialog then asks the user whether they would like a receipt, and if so how it should be delivered. At this point in the dialog we know that the client side indicated that it has gathered all the data needed and that the values are in the client <code>store</code>, and we have received confirmation from the user, so now we use <code>action</code> to call the client side implemenation for the payment_method to actually perform the transaction:
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




