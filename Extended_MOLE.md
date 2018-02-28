# A Plan for Extended MOLE

The extended MOLE should perform following tasks:

* Allow environment centric sensing, such as getting tempreture from sensors (original MOLE)
* Allow human centric sensing, such as taking photo
* Allow data evaluation on the edge
* Provide possiblity and incentive cost optimization and estimation

## TODO

* Need to specify unique participant, such that a data collector cannot evaluate his own data.

## Definition

### Global Input

* Incentive cost: In dollars
* Incentive mechanism: Fixed price or reverse auction
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
  select.device: "Mobile Phone"       # Should we specify system version and hardware parameters?
  select.minimumVersion: "Android 4.0"
  select.minimumQuality: "1440*900"
  info.humanInvolved: True
  info.location: "US"
  info.instruction: None
  info.title: "Take Photo"
  data.return: {
    image: [jpeg, png],
    tag: [String, Integer]
  }
}
```

```
MS EvaluatePhoto {
  select.device: "Mobile Phone" 
  select.minimumVersion: "Android 4.0"
  select.minimumQuality: "1440*900"
  info.humanInvolved: True
  info.location: "US"
  info.instruction: None
  info.title: "Evaluate Photo"
  data.require: {
    image: [jpeg, png],
    tag: [String, Integer]
  }
  data.return: {       # I want it returns evaluated data
    image: [jpeg, png],
    tag: [String, Integer]
  }
}
```

## Taking Photo for Vehicle Recognition Example

```
Service RecognizeVehicle {
	
  incentiveCost: 1000
  incentiveMechanism: FixedPrice
  duration: 30d  // What format of time?
	
  MS: TakePhoto {
    select.minimumVersion: "Android 4.4"
    info.location: [“US.Virginia”, “US.Washington_DC”]
    info.instruction: “./README.xml” // A format that can be displayed on Android
    info.title: “Take photo of vehicles”
    data.return: {
      image: {
        vehicle: jpeg
      },
      tag: {
        Make: String
        Model: String
        Year: String
      }
    }
    on.success: EvaluatePhoto(image, tag); exit
    on.fail: exit
  }

  MS: EvaluatePhoto {
    info.instruction: “./README.xml”
    info.title: “Evaluate photo of vehicles”
    data.require: {
      image: {
        vehicle: jpeg
      },
      tag: {
        Make: String
        Model: String
        Year: String
      }
    }
    data.return: {
      image: {
        vehicle: jpeg
      },
      tag: {
        Make: String
        Model: String
        Year: String
      }
    }
    on.success: DeepLearningTraining(image, tag); exit
    on.fail: exit
  }

  MS: DeepLearningTraining {
    info.code: “./training.tar”
    info.driver: "./train.py"
    data.size: 1000
    data.require: {
      image: {
        vehicle: jpeg
      },
      tag: {
        Make: String
        Model: String
        Year: String
      }
    }
    data.return: {
      model: PyTorch
    }
    on.success: ret model; exit
    on.fail: exit
  }
  
}

```

## EBNF of Extended MOLE

```
<Service_Suite> ::= <Service_Identity> <Service_Description>
<Service_Identity> ::= "Service " String
<Service_Description> ::= "{" {<Service_Parameter>} <Microservice_Invocations> {<Microservice_Invocations> "}"
<Service_Parameter> ::= <Parameter_Name> ": " <Parameter_Value>

<Microservice_Invocations> ::= "MS: " <MS_Identity> "{" [<MS_Detail>]{<MS_Detail>} "}"
<MS_Identity> ::= String
<MS_Detail>::= <Device_Selection>|<Information_Params>|<Data_Params>|<After_Execution_Rules>

<Device_Selection> ::= <Device_Selection_Device>|<Device_Selection_Version>|<Device_Selection_Quality>
<Device_Selection_Device> ::= "select.device: " String
<Device_Selection_Version> ::= "select.minimumVersion: " String
<Device_Selection_Quality> ::= "select.minimumQuality: " String

<Information_Params> ::= <Information_Params_Human>|<Information_Params_Location>|<Information_Params_Instruction>|<Information_Params_Title>
<Information_Params_Human> ::= "info.humanInvolved: " Boolean
<Information_Params_Location> ::= "info.location: " <L
```
