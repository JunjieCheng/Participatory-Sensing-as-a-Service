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
basic MS TakePhoto {
  select.device.is("Mobile_Phone")
  info.instruction.from("FileName.xml")
  info.title.from("Title")
  
  data.return([JPEG, PNG] image)
}
```

```
MS GetText {
  // Required
  select.device.is("Mobile_Phone")
  
  info.instruction.from("FileName.xml")
  info.title.from("Title")
  
  MS.return([String, Integer, Float] text)
}
```

```
MS base TakePhoto{
	device.select.has("camera");
	return.type = "JPEG, PNG";
	return.tag
}

MS EvaluateImage extends TakePhoto {
  // Data will be sent to two devices for evaluation. If the result of two devices are different, it will be sent to a device with high reputation.
  select.location.is("xxx")
  
  // Required
  info.instruction.from("FileName.xml")
  info.title.from("Title")
  
  MS.require([JPEG, PNG] image)
  MS.return([JPEG, PNG] image)
}
```

## Taking Photo for Vehicle Recognition Example

```
Service RecognizeVehicle {
	
  global.incentiveCost.is("1000")
  global.incentiveMechanism.is("FixedPrice")
  global.expiration.is("23:00:00 04/05/2018")
  global.numberOfDate.is("1000")
	
  MS: CollectVehiclePhoto extends MobilePhone.TakePhotoWithTag {
    select.system.is("Android")
    select.verison.greaterThanOrEq("4.4")
    select.location.is("US")
    select.location.isNot("[US.Virginia, US.Washington_DC]")
    
    info.instruction.from(“./README.xml”)
    info.title.from(“Dataset of vehicles”)
        
    on.success: ret JPEG image, String tag
    on.fail: exit
  }
  
  MS: EvaluateVehiclePhoto extends MobilePhone.EvaluatePhotoWithTag {
    select.system.is("Android")
    select.verison.greaterThanOrEq("4.4")
    select.device.differentAs("CollectVehiclePhoto")
    
    info.instruction.from(“./README.xml”)
    info.title.from(“Evaluate dataset of vehicles”)
    
    data.require(image, tag)
    
    on.success: ret JPEG image, String tag
    on.fail: exit
  }
 
}

```

## Example: Rewrite getTemp

```
Service GetTemp {

  Expiration: 21:00:00 03/06/2018
  NumberOfData: 1

  MS: ReadTempFromSensor extends Device.ReadTempreture {
    select.location.is("Nearby")
    
    on.success: ret String temp
    on.fail: getTempByLocation
  }
  
  MS: GetTempByLocation extends Internet.ReadTempreture {
    select.location.is(location)
    select.try.lessThanOrEq("3")
    
    on.success: ret String temp
    on.fail: exit
  }
  
  MS: GetLocationByGPS extends MobilePhone.ReadGPSLocation {
    select.location.is("Nearby")
    
    on.success: ret String location
    on.fail: exit
  }
    
  MS: GetLocationByCellID extends Device.ReadCellTowerLocation { 
    select.location.is("Nearby")
    
    on.success: ret String location
    on.fail: exit
  }
  
}
  
```

## Example: Air Quality Monitering

```
Service AirQualityMonitering {

  Expiration: 21:00:00 03/06/2018
  NumberOfData: 1

  MS: GetAirQualityFromSensor extends Device.ReadPM2.5 {
    select.location.is("Nearby")
    
    on.success: ret String airQuality
    on.fail: GetAirQualityFromPhoto
  } 
  
  MS: GetAirQualityFromPhoto extends Device.AnalyzePM2.5FromPhoto {
    select.input.from(image)
    
    on.success: ret String airQuality
    on.fail: exit
  }
  
  MS: TakePhoto extends MobilePhone.TakePhoto {
    select.location.is("Nearby")
    info.instruction.from(“./README.xml”)
    info.title.from(“Take Photo for Analyzing PM2.5”)
    
    on.success: ret JPEG image
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
