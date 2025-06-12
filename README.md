<p align="center">
  <img src="https://github.com/Ationet/ationetdevconnect/raw/main/Content/Images/LogoDEVCONNECT300x317.png" />
</p>

## Authorization Engine
> This API is responsible for processing all transactions that come from the different capture devices,
> whether for the Fleet, Loyalty, Gift Card or Consumer Card module.

- [Native Transaction Protocol API](AN-Native_Protocol_Integration.md)
- [Native Transaction Protocol API Specifications](AN-Native_Transaction_Protocol-Spec.md)
- [Native Transaction Protocol API messages](AN-Native_Auth_Protocol_Messages.md)

## Fleet Mobile Payment Module
> This API is responsible for processing and orchestrating all the flow that requires an MPPA type integration.

- [Fleet Mobile Payment APIs Specifications](FleetMobilePayment.md)

## Entities
> This API is responsible for interacting with all ATIONET entities, either to create or modify them, whether they are sites, companies, contracts, projects, etc.
> This API is typically used when you want to make a proprietary portal that replaces the ATIONET standard,
> or to develop a proprietary mobile app that displays this information. This document is in Swagger format.
- [Entities API Specifications](http://api.ationet.com/Help)

## Interfaces
> This API is responsible for implementing specific messages for integrations with ERP, these messages have more logic and processing than a typical standard CRUD operation message.
- [Interface API Specifications](AN-Native_Interface_Protocol-Spec.md)

## Inventory Module
> This API is responsible for interacting with the inventory module. You will be able to send telemetry information from an ATG to ATIONET.
> Information such as inventories, deliveries, alarms and product and water levels.
- [FMS API Specifications](AN-Native_Inventory_Protocol-Spec.md)

## Wallet Module
> This API focuses on exposing methods that allow implementing a wallet for the B2C segment. This API is the link between the gas station forecourt, the wallet and the processor
- [Wallet API Specifications](AN-Native_Wallet_Protocol-Spec.md)

## Consumer Card Module
> These APIs are intended to support the Consumer Card module, with CC API, you can create custom portals, custom applications, among other things. The processing of transactions goes through the CC Native Protocol API.
- [Consumer Card Native Protocol](AN-Native_ConsumerCard.md)
- [Consumer Card API Specifications](AN-Consumer_Card_API-Spec.md)

## Loyalty Module
> This API aims to process Loyalty module transactions. Whether accumulations, redemptions, etc.
- [Loyalty Native Protocol API description](AN-Native_Loyalty_Protocol-Spec.md)

## Terminal Management Module
> This API aims to expose the necessary messaging so that any device that wants to connect with Terminal Management can do so, both for monitoring, parameterization or updating.
- [Terminal Management API description](AN-Native_DeviceUpdater_Protocol-Spec.md)

## Tracking Module
> This API is responsible for interacting with the tracking module. You will be able to send geolocation information from any on board device installed in the vehicle. 
- [Tracking Interface API Protocol Specification](AN-Native-Tracking_Protocol-Spec.md)

## Fiscal Module
> This API aims to expose the necessary messaging for countries where there is specific legislation for electronic invoices. For each country there could be differences, request specific documentation.

- [Fiscal API description](AN-Fiscal_API-Spec.md)

## .NET SDK
> This documentation details how the SDK for .NET technology should be consumed. This SDK simplifies and reduces complexity for the programmer. It also describes how to install it from nuget.
- [.NET SDK Reference](AN-SDK-Reference.md)

