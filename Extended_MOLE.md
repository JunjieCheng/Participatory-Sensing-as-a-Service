# A Plan for Extended MOLE

The extended MOLE should perform following tasks:

* Allow environment centric sensing, such as getting tempreture from sensors (original MOLE)
* Allow human centric sensing, such as taking photo
* Allow data evaluation on the edge
* Provide possiblity and incentive cost optimization and estimation

## New Features

### Base Microservice

Base microservice performs a specific and basic functionality, such as getting image, getting text. Users are allowed to extend multiply basic microservice to customize their own microservices. 

In multiple inheritance, same parameters in the basic microservices will be merged. 

### Data Type

Data type indicates the type of data that the user want to collect from the microservice. It is necessary to type of data processing microservice, because the program runs on the specific devices require specific data type. 

The data type that need to specify includs String, Integer, Float etc... and file type includs image, audio and video. 

### Location

Location indicates the location of the device. It is important when the user only wants to collect data from a specific area.

### Number of Data

The original MOLE doesn't indicate the number of data to collect. It assumes that only one data will be needed. However, when a MS requires 100 data from a MS that can only privide 1 data. The syntax becomes insufficient.

The solution in this version is to indicate the number of data required in the return and require field. It looks like an array.

### Additional Information

The original MOLE assumes that the MS is not human involved. Therefore, no additional information will be needed for sensors except the commands from the executor. However, when the executor is human, the user must provide additional information to guide participants to collect correct type of data. The additional information can be examples of correct data and incorrect data.

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

## Base Microservice Example

```
MS GetImage {
  // Required
  info.instruction.from("FileName.xml")
  info.title.is("Title")
  
  data.return([JPEG, PNG] image)
}
```

```
MS GetText {
  // Default
  select.device.is("Mobile_Phone")
  select.system.is("Android")
  select.verison.greaterThanOrEq("4.4")
  select.location.is("US")
  
  // Required
  info.instruction.from("FileName.xml")
  info.title.is("Title")
  
  MS.return([String, Integer, Float] text)
}
```

```
MS EvaluateImage {
  // Data will be sent to two devices for evaluation. If the result of two devices are different, it will be sent to a device with high reputation.
  // Default
  select.device.is("Mobile_Phone")
  select.system.is("Android")
  select.verison.greaterThanOrEq("4.4")
  select.location.is("US")
  
  // Required
  info.instruction.from("FileName.xml")
  info.title.is("Title")
  
  MS.require([JPEG, PNG] image)
  MS.return([JPEG, PNG] image)
}
```

## Taking Photo for Vehicle Recognition Example

```
Service RecognizeVehicle {
	
  incentiveCost: 1000
  incentiveMechanism: FixedPrice
  duration: 30d  // What format of time?
	
  MS: CollectVehiclePhoto {
    select.device.is("Mobile_Phone")
    select.system.is("Android")
    select.verison.greaterThanOrEq("4.4")
    select.location.is("US")
    select.location.isNot("[US.Virginia, US.Washington_DC]")
    
    info.instruction.from(“./README.xml”)
    info.title.is(“Dataset of vehicles”)
    
    GetImage.return(JPEG vehicle[3])
    GetText.return(String make, String model, String year)
    
    on.success: EvaluatePhoto(vehicle, make, model, year); exit
    on.fail: exit
  }
  
  MS: EvaluateVehiclePhoto {
    select.device.is("Mobile_Phone")
    select.system.is("Android")
    select.verison.greaterThanOrEq("4.4")
    select.location.is("US")
    
    info.instruction.from(“./README.xml”)
    info.title.is(“Evaluate dataset of vehicles”)
    
    EvaluateImage.require(JPEG vehicle[10][3])
    EvaluateImage.return(JPEG vehicle[10][3])
    EvaluateText.require(String make[10], String model[10], String year[10])
    EvaluateText.return(String make[10], String model[10], String year[10])
    
    on.success: DeepLearningTraining(vehicle[3], make, model, year); exit
    on.fail: exit
  }

  MS: DeepLearningTraining extends MachineLearning {
    info.code.from(“./training.tar”)
    info.driver.from("./train.py")
    
    data.require(JPEG vehicle[1000][3], String make[1000], String model[1000], String year[1000])
    data.return(PyTorch model)
   
    on.success: ret model; exit
    on.fail: exit
  }
  
}

```

## 

## EBNF of Extended MOLE

```
<Service_Suite> ::= <Service_Identity> <Service_Description>
<Service_Identity> ::= "Service " <String>
<Service_Description> ::= "{" {<Service_Parameter>} <Microservice_Invocations> {<Microservice_Invocations> "}"
<Service_Parameter> ::= <Parameter_Name> ": " <Parameter_Value>

<Microservice_Invocations> ::= "MS: " <MS_Identity> "{" [<MS_Detail>]{<MS_Detail>} "}"
<MS_Identity> ::= <String>
<MS_Detail>::= <Device_Selection>|<Information_Params>|<Data_Params>|<After_Execution_Rules>

<Device_Selection> ::= <Device_Selection_Device>|<Device_Selection_Version>|<Device_Selection_Quality>
<Device_Selection_Device> ::= "select.device: " <String>
<Device_Selection_Version> ::= "select.minimumVersion: " <String>
<Device_Selection_Quality> ::= "select.minimumQuality: " <String>

<Information_Params> ::= <Information_Params_Human>|<Information_Params_Location>|<Information_Params_Instruction>|<Information_Params_Title>
<Information_Params_Human> ::= "info.humanInvolved: " <Boolean>
<Information_Params_Location> ::= "info.location: " (<Location>|("[" <Location>{", " <Location>} "]"))
<Location> ::= <Country_Code> ["." <State_Name> ["." <City_Name>]]
<Information_Params_Instruction> ::= "info.instruction: " <File_Path>
<Information_Params_Title> ::= "info.title: " <String>

<Data_Params> ::= <Data_Params_Require>|<Data_Params_Return>
<Data_Params_Require> ::= "data.require: {" <Data_Params_Category>{"," <Data_Params_Category>} "}"
<Data_Params_Return> ::= "data.return: {" <Data_Params_Category>{"," <Data_Params_Category>} "}"
<Data_Params_Category> ::= <Data_Params_Category_Name> ": {" <Data_Parameter>{<Data_Parameter>} "}"
<Data_Parameter> ::= <Data_Parameter_Name> ": " <Data_Parameter_Type>
<Data_Parameter_Name> ::= <String>
<Data_Parameter_Type> ::= <Data_Type>|<File_Type>

<After_Execution_Rules> ::= "on." <Condition> ":" (<Redirection>|<Return>){"; " (<Redirection>|<Return>)}
<Condition> ::= "success"|"fail"
<Redirection> ::= (<MS_Identity> "(" (<Data_Parameter_Name>|"_"){", " (<Data_Parameter_Name>|"_")} ")")|"exit"
<Return> ::= <Data_Parameter_Name>
```
