# A Plan for Extended MOLE

The extended MOLE should perform following tasks:

* Allow environment centric sensing, such as getting tempreture from sensors (original MOLE)
* Allow human centric sensing, such as taking photo and evaluating photo
* Provide possiblity and incentive cost optimization and estimation (SLA)

## New Features

### Basic Microservice

Basic microservice performs a specific and basic functionality, such as taking photo, getting text. Each customized microservice must extend one basic microservice.

### Data Type

Data type indicates the type of data that the user want to collect from the microservice. It is necessary to type of data processing microservice, because the program runs on the specific devices require specific data type. The data type that need to specify includs String, Integer, Float etc... and file type of image, audio and video. 

The data type needs to be specified when return the result in the format "<DataType> <VariableName>".

### Location

Location indicates the location of the device. It is important when the user only wants to collect data from a specific area. The location will be specified in the format "select.location.[is|isNot]("US.[State|City].[City]")". The default location is US. Is and isNot can overlap to exclude some place, but each of they can only appear at most once.

There are three kinds of location now: city name, latitude and longitude, and access point id.

When selecting city name, it includes all access point in this city. When selecting latitude and longitude, user must indicate the range around this point. When selecting access point id, the default range is the covering range of the access point.

### Number of Data

This indicates the number of data that user wants to collect. The current solution of global input looks insufficient because it becomes confused when the data is merged in the control flow. 

### Additional Information

The original MOLE assumes that the MS is not human involved. Therefore, no additional information will be needed for sensors except the commands from the executor. However, when the executor is human, the user must provide additional information to guide participants to collect correct type of data. The additional information can be examples of correct data and incorrect data.

## TODO

* Server and access point
* Multiple data type return
* Multiple data return
* Location expression
* Location type: city name, latitude and longitude, and access point id.
* Format of global input
* Is the syntax consistant? How to differentiate is, isNot, eq, greaterThan etc.?

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

## Global Input

```
global.incentiveCost = "1000"
global.expiration = "23:00:00 04/05/2018"
global.numberOfData = "1000"
global.location = ["US.Virginia", "US.Washinton.DC"]
global.reward = "FixedPrice"/"ReverseAuction"
```

## Base Microservice Example

```
basic MS: TakePhoto {
  select.device = "Mobile_Phone"
  select.device.differentAs()
  select.device.sameAs()
  select.version = "4.0+"
  select.user.reputation = "30+"
  
  info.instruction = "FileName.xml"
  info.title = "Title"
  info.reward = "1"
  
  return [JPEG, PNG] image
}
```

## Example: City Health
```
Service CityHealth {
  
  global.incentiveCost = "1000"
  global.expiration = "23:00:00 04/05/2018"
  global.numberOfData = "1000"
  global.location = ["US.Virginia", "US.Washinton.DC"]
  
  MS: TakePhoto() extends MobilePhone.TakePhotoWithTag {
    select.system = "Android"
    select.version = "4.4+"
    
    info.instruction = “./README.xml”
    info.title = “City Health”
    info.reward = "0.5"
    
    on.success: EvaluatePhoto(JPEG image, String tag)
    on.fail: exit
  }
  
  MS: TakePhoto2() extends MobilePhone.TakePhotoWithTag {
    select.system = "Android"
    select.version = "4.4+"
    select.user.reputation = "70+"
    
    info.instruction = “./README.xml”
    info.title = “City Health”
    info.reward = "0.5"
    
    on.success: return JPEG image, String tag
    on.fail: exit
  }
  
  MS: EvaluatePhoto(JPEG image, String tag) extends MobilePhone.EvaluatePhotoWithTag {
    select.system = "Android"
    select.verison = "4.4+"
    select.device.differentAs("TakePhoto")
    
    info.instruction = “./README.xml”
    info.title = “Evaluate City Health”
    info.reward = "0.2"
        
    on.success: return JPEG image, String tag
    on.fail: exit
  }

}
```

## Example: Rewrite getTemp

```
Service GetTemp {

  global.expiration = "21:00:00 03/06/2018"
  global.numberOfData = 1
  global.location = ["US.Virginia.Blacksburg"]

  MS: ReadTempFromSensor() extends Device.ReadTempreture {    
    on.success: return String temp
    on.fail: GetTempByLocation(_)
  }
  
  MS: GetTempByLocation(String location) extends Internet.ReadTempreture {
    select.try.lessThanOrEq("3")
    
    on.success: return String temp
    on.fail: exit
  }
  
  MS: GetLocationByGPS() extends MobilePhone.ReadGPSLocation {    
    on.success: GetTempByLocation(String location)
    on.fail: exit
  }
    
  MS: GetLocationByCellID() extends Device.ReadCellTowerLocation {     
    on.success: GetTempByLocation(String location)
    on.fail: exit
  }
  
}
  
```

## Example: Air Quality Monitering

```
Service AirQualityMonitering {

  global.expiration = "21:00:00 03/06/2018"
  global.numberOfData = 1
  global.location = ["US.Virginia.Blacksburg"]
  
  MS: GetAirQualityFromSensor() extends Device.ReadPM2.5 {    
    on.success: return String airQuality
    on.fail: GetAirQualityFromPhoto(_)
  } 
  
  MS: GetAirQualityFromPhoto(JPEG image) extends Device.AnalyzePM2.5FromPhoto {    
    on.success: return String airQuality
    on.fail: exit
  }
  
  MS: TakePhoto() extends MobilePhone.TakePhoto {
    info.instruction.from(“./README.xml”)
    info.title.from(“Take Photo for Analyzing PM2.5”)
    
    on.success: GetAirQualityFromPhoto(JPEG image)
    on.fail: exit
  }
 
}
```

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
