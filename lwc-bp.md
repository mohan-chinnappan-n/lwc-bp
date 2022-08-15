## LWC Bundle nameing convention 
- Markup
	- Use **camelCase** to name the component
	- Use **kehab-case** to reference a component 
	- Should begin with a lower case e.g. ```helloWorld.html```
	- Can contain alphanumeric or underscore characters
	- Can't contain hypen
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


- JS 
	- Class name should be in **PascalCase**
```
import { LightningElement, api, wire } from 'lwc';

export default class WireGetRecordAccount extends LightningElement {
}

```
---
