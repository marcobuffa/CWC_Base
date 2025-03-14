# CWC_Base

## A very basic (& useless) Custom Web Control (CWC) to show how stuff works with WinCC Unified

![A graduation hat]({d722178a-239f-4be0-ab55-b24616547bd7}/assets/learning.png)

Icon created by [Tanah Basah - Flaticon](https://www.flaticon.com/free-icons/course")

## ToDo:

- [x] Get requests by customers
- [x] Get requests by colleagues
- [x] Find a timeslot
- [x] Write code
- [ ] Debug
- [ ] Test on a Unified Comfort Panel
- [ ] Debug again
- [ ] Write a meaningful documentation

## How to create a CWC from this stuff:

1. Download/pull/whatever the code
2. cd in the **{d722178a-239f-4be0-ab55-b24616547bd7}** folder
3. Create **{d722178a-239f-4be0-ab55-b24616547bd7}.zip** file with the **{d722178a-239f-4be0-ab55-b24616547bd7}** folder content
> [!IMPORTANT]
> Do not include the **{d722178a-239f-4be0-ab55-b24616547bd7}** folder itself as root folder in the file zip!
4. Copy the **{d722178a-239f-4be0-ab55-b24616547bd7}.zip** file in **C:\Program Files\Siemens\Automation\Portal V1x\Data\Hmi\CustomControls** folder
5. Refresh the **My Controls** right side pane in the TIA Portal WinCC Unified screen editor
6. Enjoy

## Documentation:

### How does this works?
This CWC (Custom Web Control) is intended to show:
1. the 3 data exchange mechanisms between a WinCC Unified RT and a CWC
2. how to discriminate between CWC being executed in a browser, in TIA Portal (yes, TIA Portal does execute CWCs) or in a WinCC Unified RT

### Data exchange
A CWC is just a web page running inside an iframe in a WinCC Unified project.

Any client-side code can be run in a CWC, but this would be nearly useless without any data exchange mechanisms between the inner content of the CWC and the WinCC Unified RT (outside the CWC).

3 mechanisms are available to get a fully bidirectional data exchange: parameters, events and methods.

#### Parameters
This is the simplest method: number, string, or structured (not shown in this example) variables can be exchanged between WinCC Unified and the CWC. They are defined in the [manifest.json file]({d722178a-239f-4be0-ab55-b24616547bd7}/manifest.json) (see the "properties" section):

```
"properties": {
    "NumberParameter":{
        "type": "number",
        "default": 0
    },
    "StringParameter":{
        "type": "string",
        "default": ""
    }
}
```

This is a bidirectional mechanism: a parameter can be written by WinCC Unified RT and read by the CWC, or viceversa.

A callback mechanism is available inside the CWC in order to get notified if a new value is written by WinCC Unified (see [index.html]({d722178a-239f-4be0-ab55-b24616547bd7}/control/index.html)):

```
function dataFromWinCC(data){
    console.log("Nuovo parametro ricevuto da WinCC:");
    console.log(data);
    switch (data.key) {
        case "NumberParameter":
                document.getElementById('NumberParameter').value = data.value;
            break;
        case "StringParameter":
                document.getElementById('StringParameter').value = data.value;
            break;
    }
}
```

And, later, the registration:

```
WebCC.onPropertyChanged.subscribe(dataFromWinCC);
```

Sending a new value to WinCC Unified is even simpler (see [index.html]({d722178a-239f-4be0-ab55-b24616547bd7}/control/index.html)):

```
WebCC.Properties.NumberParameter = document.getElementById('NumberParameter').value;
WebCC.Properties.StringParameter = document.getElementById('StringParameter').value;
```

#### Events
Events can be generated by the CWC and the associated function in WinCC Unified can be projected like any standard event generated by any standard control in TIA Portal: see Properties > Events of the CWC control in the low pane of screen engineering in TIA Portal, selecting the CWC instance in the page.

This is a one-direction mechanism: data can be exchanged from the CWC to WinCC Unified RT.

The [manifest.json file]({d722178a-239f-4be0-ab55-b24616547bd7}/manifest.json) shall list all available events of the CWC (see the "events" section):

```
"events": {
    "EventWithNumber": {
        "arguments": {
            "NumberParameter": {
                "type": "number"
            }
        },
        "description": "Just an event delivering a number parameter"
    },
    "EventWithString": {
        "arguments": {
            "StringParameter": {
                "type": "string"
            }
        },
        "description": "Just an event delivering a string parameter"
    }
}
```

As you can see, events can carry parameters too. Please, use just one, non-structured parameter per event. If you need a structure, just use a (json) string.

Firing an event from the CWC itself is pretty simple (see [index.html]({d722178a-239f-4be0-ab55-b24616547bd7}/control/index.html)):

```
WebCC.Events.fire("EventWithNumber", document.getElementById('NumberParameter').value);
```

#### Methods
Methods are the opposite of events: one-way direction (from WinCC Unified to TIA Portal), with an optional (one, non-structured) parameter.

Calling a CWC method from WinCC Unified is a little bit more tricky. You need to create a JS script in the screen engineering of TIA Portal and associate something like this:

```
export function Button_3_OnTapped(item, x, y, modifiers, trigger) {
    Screen.Items("Base CWC 4 training_1").MethodWithNumber(47);
}
```

This example just calls the CWC's MethodWithNumber passing the value 47 as parameters. In this case, this is fired by tapping a standard button in WinCC Unified, but you can associate the calling function to any event in the screen execution context of WinCC Unified.

Dealing with methods inside the CWC is also pretty simple (see [index.html]({d722178a-239f-4be0-ab55-b24616547bd7}/control/index.html)):

```
methods: {
    MethodWithNumber: function(NumberParameter){
        console.log("Chiamato metodo MethodWithNumber con parametro: " + NumberParameter);
        document.getElementById('Method').innerHTML = "L'ultimo metodo chiamato è stato: <em>MethodWithNumber</em>"
        document.getElementById('NumberParameter').value = NumberParameter;
    },
    MethodWithString: function(StringParameter){
        console.log("Chiamato metodo MethodWithString con parametro: " + StringParameter);
        document.getElementById('Method').innerHTML = "L'ultimo metodo chiamato è stato: <em>MethodWithString</em>"
        document.getElementById('StringParameter').value = StringParameter;
    }
},
```

### Discriminate between environments
Being able, from the CWC perspective, to discriminate the outer execution environment can be useful in order to:
- make the debugging process simpler
- test the CWC in a browser
- force parameters or appareance in engineering
- ...

The WebCC library makes this quite easy. Have a look a the [index.html]({d722178a-239f-4be0-ab55-b24616547bd7}/control/index.html):

```
WebCC.start(
    function(result) {
        if (result) { //c'è un'istanza di WinCC in esecuzione
            console.log("Connessione con WinCC avvenuta con successo");
            if (WebCC.isDesignMode) { //l'istanza è di ingegneria (TIA Portal)
                document.getElementById('Connection').innerHTML = "Questo CWC è in esecuzione in <em>ingegneria (TIA Portal)</em>";
            } else { //l'istanza è di runtime
                document.getElementById('Connection').innerHTML = "Questo CWC è in esecuzione nel <em>runtime</em>";
                try {
                    wincc = true; //attiva flag presenza runtime
                    WebCC.onPropertyChanged.subscribe(dataFromWinCC); //registrazione callback chiamata quando arrivano nuovi dati da WinCC
                } 
                catch (error) {
                    console.log(error);
                }
            }
            
        } else {
            console.log("Connessione con WinCC non riuscita (sono in un browser?)");
        }
    },
```

The `WebCC.start` method returns true if a generic WinCC Unified rendering engine can be contacted. A false return value means the CWC is executed in a browser, just opening the index.html entry file.

If a true value is returned, the `WebCC.isDesignMode` can be used to discriminate between TIA Portal rendering (true is returned) or a real WinCC Unified RT rendering engine (false is reported).
