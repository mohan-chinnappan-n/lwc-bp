# LWC  Naming convention 
## Markup
- Use **camelCase** to name the component e.g: ```helloWorld```
- Use **kehab-case** to reference a component 
- Should begin with a lower case e.g. ```helloWorld.html```
- Can contain alphanumeric or underscore characters
- Can't contain (-) 

```
sfdx force:lightning:component:create -n "Wrong-Name3" --type lwc -d force-app/main/default/lwc
ERROR running force:lightning:component:create:  Name must contain only alphanumeric characters.
```

- Can't contain white spaces (blanks)

```
sfdx force:lightning:component:create -n "Wrong Name3" --type lwc -d force-app/main/default/lwc
ERROR running force:lightning:component:create:  Name must contain only alphanumeric characters.
```

- Can't end with underscore (_)
```
sfdx force:lightning:component:create -n WrongName2_ --type lwc -d force-app/main/default/lwc
ERROR running force:lightning:component:create:  Name can't end with an underscore.
```
- Can't have more than one consecutive underscores
```
sfdx force:lightning:component:create -n "Wrong__Name3" --type lwc -d force-app/main/default/lwc
ERROR running force:lightning:component:create:  Name can't contain 2 consecutive underscores.
```
- Folder and files names must have same prefix name

### Correct name
```
sfdx force:lightning:component:create -n helloWorld --type lwc -d force-app/main/default/lwc 
```
```
target dir = /Users/mchinnappan/bp/force-app/main/default/lwc
   create force-app/main/default/lwc/helloWorld/helloWorld.js
   create force-app/main/default/lwc/helloWorld/helloWorld.html
   create force-app/main/default/lwc/helloWorld/__tests__/helloWorld.test.js
   create force-app/main/default/lwc/helloWorld/helloWorld.js-meta.xml
```


### Wrong name - how lwc generator fixes it
```
sfdx force:lightning:component:create -n WrongName --type lwc -d force-app/main/default/lwc
```
```
target dir = /Users/mchinnappan/bp/force-app/main/default/lwc
   create force-app/main/default/lwc/wrongName/wrongName.js
   create force-app/main/default/lwc/wrongName/wrongName.html
   create force-app/main/default/lwc/wrongName/__tests__/wrongName.test.js
   create force-app/main/default/lwc/wrongName/wrongName.js-meta.xml
```


## JS 
- Class name should be in **PascalCase**

```js
import { LightningElement, api, wire } from 'lwc';
export default class WireGetRecordAccount extends LightningElement { }

```
---

# Calling Apex class (```@salesforce/apex/```)

## Wiring service
- The wire service provisions an **immutable stream of data** to the component. Each value in the stream is **a newer version** of the value that precedes it.

- Supports reactive variables, which are prefixed with ```$```
- If a reactive variable changes, the wire service **provisions** new data. 
	- provisions means
		- Instead of “requests” or “fetches” if the data exists in the client cache, it will be used for this
### Note
- The wire service delegates control flow to the **Lightning Web Components engine**. 
- Delegating control is great for **read** operations, but it isn’t great for create, update, and delete operations. 
- As a developer, you want complete control over operations that change data. That’s why you perform create, update, and delete operations with a JavaScript API (like [lightning/uiRecordApi](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.reference_lightning_ui_api_record)) instead of with the wire service.


### Imperative 

```html 
<!-- apexImperativeMethod.html -->
<template>

<p class="slds-var-m-bottom_small">
	<!-- onclick is handled by imperative method: handleLoad -->
 	<lightning-button label="Load Contacts" onclick={handleLoad} ></lightning-button>
</p>
</template>

```

```js
// apexImperativeMethod.js

import { LightningElement } from 'lwc';
import getContactList from '@salesforce/apex/ContactController.getContactList';

// Modules scoped with @salesforce add functionality to Lightning web components at runtime.



export default class ApexImperativeMethod extends LightningElement {
    contacts;
    error;

    handleLoad() { // Imperative method
        getContactList()
            .then((result) => {
                this.contacts = result;
                this.error = undefined;
            })
            .catch((error) => {
                this.error = error;
                this.contacts = undefined;
            });
    }
}

```

```java 
// ContactController.cls
public with sharing class ContactController {
	@AuraEnabled(cacheable=true)
	// marking actions as storable (cacheable) 
	// allows us to quickly show cached data from client-side storage 
	// without waiting for a server trip. 
	//  - Never mark as storable an action that updates or deletes data.

	// Client-side storage is automatically configured in Lightning Experience and the Salesforce mobile app.
	/*
	If the cached data is stale, the framework retrieves the latest data from the server. 
	Caching is especially beneficial for users on high latency, slow, or unreliable connections such as 3G networks.
	
	Most server requests are read-only and idempotent, which means that a request can be repeated or retried as often as necessary without causing data changes

	The responses to idempotent actions can be cached and quickly reused for subsequent identical actions. For storable actions, the key for determining an identical action is a combination of:

	Apex controller name
	Method name
	Method parameter values


	*/
    public static List<Contact> getContactList() {
		// https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_with_security_enforced.htm
        return [
            SELECT Id, Name, FirstName, LastName, Title, Phone, Email, Picture__c FROM Contact
			WHERE Picture__c != NULL
			WITH SECURITY_ENFORCED
            LIMIT 10
        ];
    }
```


### Wire

```html
 <template if:true={record.data}>
	<div> <pre>{recordStr}</pre> </div>
 </template>
```

- Syntax
```
import { adapterId } from 'adapterModule';
// import { getRecord } from 'lightning/uiRecordApi';

@wire(adapterId, adapterConfig)
propertyOrFunction;

/*
//adapterId  -> getRecord
//adapterConfig ->  { recordId: '$userId', fields: [NAME_FIELD], optionalFields: [EMAIL_FIELD] } 

 @wire(getRecord, {
        recordId: '$userId', // <--- reactive variable
        fields: [NAME_FIELD],
        optionalFields: [EMAIL_FIELD]
    })
    record; // propertyOrFunction 
*/

```
```js
import { LightningElement, wire } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';
// https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.reference_wire_adapters_record
import NAME_FIELD from '@salesforce/schema/User.Name';
import EMAIL_FIELD from '@salesforce/schema/User.Email';
import Id from '@salesforce/user/Id';

export default class WireGetRecord extends LightningElement {
    userId = Id;

	// about reactive 
	/*
	The $ prefix tells the wire service to treat it as a property of the class and evaluate it as
	this.propertyName. 
	The property is reactive. If the property’s value changes, new data is provisioned and the component rerenders.
	*/

    @wire(getRecord, {
        recordId: '$userId', // <--- reactive variable
        fields: [NAME_FIELD],
        optionalFields: [EMAIL_FIELD]
    })
    record; //wire property

    get recordStr() {
        return this.record ? JSON.stringify(this.record.data, null, 2) : '';
    }
}

```

### Decorate a function with @wire
```js
// wireFunction.js
import { LightningElement, api, track, wire } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';

export default class WireFunction extends LightningElement {
    @api recordId;
    @track record;
    @track error;

    @wire(getRecord, { recordId: '$recordId', fields: ['Account.Name'] })
    wiredAccount({ error, data }) { // The wire service provisions the function an object with error and data properties, just like a wired property.
        if (data) {
            this.record = data; // property is assigned with the data got provisioned
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.record = undefined;
        }
    }
    get name() {
        return this.record.fields.Name.value;
    }
}
```

|Style|Notes|Comments|
|---|---|---|
|Imperatively|||
|Wire|Wire Property, Wire function|use wire over imperative method invocation. Wiring to property is preferred|


#  Lightning Data Service (LDS)

![LDS](https://resources.docs.salesforce.com/images/96c6c99f3a530fbd2600a734ee804326.png)
