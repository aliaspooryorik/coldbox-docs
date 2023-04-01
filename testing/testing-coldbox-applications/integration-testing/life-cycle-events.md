# Life-Cycle Events

![](<../../../.gitbook/assets/ColdBox Virtual App Life-Cycle.png>)

ColdBox testing leverages TestBox's testing life-cycle events ([https://testbox.ortusbooks.com/primers/testbox-bdd-primer/life-cycle-methods](https://testbox.ortusbooks.com/primers/testbox-bdd-primer/life-cycle-methods)) in order to prepare the virtual ColdBox application, request context and then destroy it. By default, a virtual application is loaded for all test cases contained within a test bundle CFC via the `beforeAll()` and destroyed under `afterAll()`.

```javascript
function beforeAll(){
    super.beforeAll();
    // do your own stuff here
}

function afterAll(){
    // do your own stuff here
    super.afterAll();
}
```

> **Important** If you override any of these methods and do not funnel the super call, then you might get cached or unexpected results.

## Improving Performance: Unloading ColdBox

The default for integration testing is that the virtual ColdBox application will be **destroyed** or **unloaded** in **each test**. To keep the virtual application **running across multiple test bundle** tests you will need to use the `unloadColdBox=false` annotation or the `this.unloadColdBox=false` setting in your `beforeAll()` method. This will stop the testing classes from destroying ColdBox and improving performance.

```javascript
component extends="coldbox.system.testing.BaseTestCase" unloadColdBox=false{
    
    this.unloadColdBox = false;
    
    function beforeAll(){
        super.beforeAll();
        // do your own stuff here
    }
}
```

