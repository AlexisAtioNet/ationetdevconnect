![ationetlogo](Content/Images/ATIOnetLogo_250x70.png) 
# Fleet Mobile Payment Documentation Center

## Overview

![ationetTR](Content/Images/SiteSystemCommander/SiteSystem_Diagram.drawio2.png)

### Introduction

This Implementation Guide is intended to guide petroleum convenience retailers and their associated vendors when implementing mobile payment solutions consistent with
ISO 12812. 
</br>
>Note: ISO 12812 is document that will provide requirements, guidance and use cases for all stakeholders in the mobile payments arena.


### Entities

This section outlines the logical entities, including location options, for Mobile Payment and identifies possible physical architectures. The term “entity” is used in this
document to differentiate logical processing functionality without regard to its physical location in an implementation. 


```Mobile Payment Application (MPA):```  This entity is a software application embedded in a Mobile Device or downloaded by a consumer onto a Mobile Device, such as a smart
phone or tablet, which enables mobile payments for in-store and forecourt transactions.

```Mobile Payment Processing Application (MPPA):``` This entity is an application provided by the Mobile Payment Processor (MPP) not on the Mobile Device that is responsible for
interfacing between the Token Vault or Token/Trusted Service Provider, the MPA, the Site System, the Payment Front End Processor (PFEP), and the Loyalty Front End Processor (LFEP) in order to authorize transactions.

```Payment Front End Processor (PFEP):``` This entity is a host that facilitates the authorization of payment transactions between the MPPA or the Site System and the
Issuer networks. The standard does not dictate the processing that is performed by the PFEP for each payment method. This entity is sometimes referred to as the Front End
Processor (FEP).

```Site System:``` This entity encompasses the site equipment and components (hardware and software) and may perform the function of local card processing business rules,
such as consumer prompting, local velocity checking and receipt formatting and printing. Examples of site systems include Point of Sale (POS), Outside Sales Processor
(OSP), Electronic Payment Server (EPS) and Forecourt Device Controller (FDC).
</br>
>Note: MOBILE PAYMENT API is the common interface through which the MPA sends and receives requests from the MPPA. The description, you can read about  the methods and how to consume in [API Section](#ATIONet-PFEP-Fleet-Mobile-Payment-Api).



## MPPA Socket Server (TCP/XML) 
- Site System - [English version](ATIONet_Mobile_Payment_Fleet_Api_-EN.md#site-system-implementation-guide)
- Payment Processor  
     - Fully Integrated (ATIONET as payment processor) [English version](ATIONet_Mobile_Payment_Fleet_Api_-EN.md#ationet-configuration)
- Dinamyc QR Mode [English version](ATIONet_Dynamic_QR_Code_Payments-EN.md) / [Versión Español](ATIONet_Dynamic_QR_Code_Payments-ES.md)
- Offline Mode [Versión Español](ATIONet_OFFLine_Payments-ES.md)


## MPPA Server Send Event (HTTP/JSON) 

- Site System  [English version](Ationet_Fleet_Mobile_Payment_SiteSystem_Implementation-EN.md) / [Versión Español](Ationet_Fleet_Mobile_Payment_SiteSystem_Implementation-ES.md)
     - PrePaid  [English version](Ationet_Fleet_Mobile_Payment_PrePaid-EN.md) / [Versión Español](Ationet_Fleet_Mobile_Payment_PrePaid-EN.md)
     - Postpaid [English version](Ationet_Fleet_Mobile_Payment_PostPaid-EN.md)  / [Versión Español](Ationet_Fleet_Mobile_Payment_PostPaid-ES.md) 
- Payment Processor  
     - Fully Integrated (ATIONET as payment processor) [English version](ATIONet_Mobile_Payment_Fleet_Api_-EN.md#ationet-configuration)
     - External Integrated (External Wallet) [English version](Ationet_Fleet_Mobile_Payment_Wallet_API-EN.md) / [Versión Español](Ationet_Fleet_Mobile_Payment_Wallet_API-ES.md)








