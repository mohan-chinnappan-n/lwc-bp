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

```html - apexImperativeMethod.html
<template>

<p class="slds-var-m-bottom_small">
                <lightning-button
                    label="Load Contacts"
                    onclick={handleLoad} 
                ></lightning-button>
            </p>

</template>

```

```js - apexImperativeMethod.js

import { LightningElement } from 'lwc';
import getContactList from '@salesforce/apex/ContactController.getContactList';

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

```java - ContactController.cls
public with sharing class ContactController {
	@AuraEnabled(cacheable=true)
    public static List<Contact> getContactList() {
        return [
            SELECT Id, Name, FirstName, LastName, Title, Phone, Email, Picture__c FROM Contact
			WHERE Picture__c != NULL
			WITH SECURITY_ENFORCED
            LIMIT 10
        ];
    }
```

|---|---|---|
|Imperatively|||
|Wire|Wire Property, Wire function|use wire over imperative method invocation.Prefer wiring to a property|
