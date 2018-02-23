# A Plan for Extended MOLE

The extended MOLE should perform following tasks:

* Allow environment centric sensing, such as getting tempreture from sensors (original MOLE)
* Allow human centric sensing, such as taking photo
* Allow data evaluation on the edge
* Provide possiblity and incentive cost optimization and estimation

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

Microservices are implemented on the gateway. They are components that provide to users. Each microservie perform a function and provide parameter for users to change.

* Location: Location of devices will be specified in this format: "US", "US.Virginia", "US.Virginia.Blacksburg", "US.Washington_DC"
* Instruction: A instruction that shows to participants about what to do. It should be a format that can be displayed on Android System.
* Title: A title that shows to participants in the task list.
* Return: It specify the result of this microservice. Result types are defined by the API, but users can specify one or more of return types, number and quality. Executor and client will be implemented to fit specific microservice and show informatioin on the client side.
* Parameter passing: Parameters will be passed by directly call the next microserver. Missing parameter will be replace by underline. The compiler will figure out the control flow.

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
