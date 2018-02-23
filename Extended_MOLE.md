# A Plan for Extended MOLE

## TODO

* Need to specify unique participant, such that a data collector cannot evaluate his own data.
* How to specify quality of data?
* Should we specify variable type?

## Definition

### Global Input

* Incentive cost: In dollars
* Incentive mechanism: Fixed price or reverse auction
* Number of data to collect
* Duration: How long does the service exist on edge devices.

### Microservice API

Microservice API defines the functionality to perform, such as taking photo. It includes required and optional parameters for users to specify. Default parameter can be overrided.

Each MS should specify exact thing to do. 

* Location: Location of devices
* Instruction: A instruction that shows to participants about what to do. It should be a format that can be displayed on Android System.
* Title: A title that shows to participants
* Return: Result of this microservice. Return type, number and quality should be specified in this field. Executor and client can be implemented to fit specific microservice.
* Parameter passing: Parameters will be passed by directly call the next microserver. Missing parameter will be specify by underline.

## Microservice API Example

```
MS TakePhoto {
  device: "Mobile Phone"       # Should we specify system version and hardware parameters?
  humanInvolved: True
  location: "US"
  instruction: None
  return: {
    image: [jpeg, jpg, png]
    tag: String
  }
}
```

```
MS EvaluatePhoto {
  device: "Mobile Phone" 
  require: [image, tag]
  humanInvolved: True
  location: "US"
  instruction: None
  return: {       # I want it returns evaluated data
    image: [jpeg, jpg, png]
    tag: String
  }
}
```

## Taking Photo for Vehicle Recognition Example

```
Service RecognizeVehicle {
	
  incentiveCost: 1000
  incentiveMechanism: FixedPrice
  numberOfData: 100
  Duration: 30d
	
  MS TakePhoto {
    location=[“US.Virginia”, “US.Washington_DC”]
    instruction=“./README.html” // A format that can be displayed on Android
    title: “Take photo of vehicles”
    return: {
      tag={
        Make: String
        Model: String
        Year: String
      }
    }
    on.success: evaluatePhoto(image, tag); exit
    on.fail: exit
  }

  MS EvaluatePhoto {
    instruction: “./README.html”
    title: “Evaluate photo of vehicles”
    on.success: DeepLearningTraining(image, tag); exit
    on.fail: exit
  }

  MS DeepLearningTraining {
    code=“./training.tar”
    on.success: ret model; exit
    on.fail: exit
  }
  
}

```
