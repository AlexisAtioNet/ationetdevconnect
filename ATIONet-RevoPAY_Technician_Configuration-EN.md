## Ationet Configuration For RevoPAY Technician

![ationetTR](Content/Images/RevoPAYTechnician/NWTechnicianRole.png)

### NWTechnician Role Configuration

In order to log in RevoPAY Technician successfully, it needs a user with the specific role of NWTechnician. 
If the introduced user doesn't have that role, it won't be able to log in. 

NWTechnician supports several networks, therefore, the same technician may be able to configure several sites, from several entities. The role only has access to the Sites, Terminals and Notifications menu, as they are the only modules the app will need.

![ationetTR](Content/Images/RevoPAYTechnician/NWTechnicianRole_NavigationMenuAtionet.png)

And once the user is ready, you can now log in RevoPAY Technician!

![ationetTR](Content/Images/RevoPAYTechnician/Technician_login.png)

<br/>
<br/>


## Site with AN-MobilePayment associated terminal

Once you have already logged with a NWTechician user, you will be asked to select wich site to configure. The shown list will ONLY show sites that belong to the selected entity and, most importantly, among those sites, ONLY the ones that have an ACTIVE AN-MobilePayment terminal associated.

If the sites display does not show your site, then please check the following validations.

* The site exists in Ationet (Check in Ationet's Sites module).
* The site has an associated AN-MobilePayment terminal (Check in Ationet's Terminals module).
* The AN-MobilePayement terminal is active.

<br/>

![ationetTR](Content/Images/RevoPAYTechnician/Ationet_site.png)

![ationetTR](Content/Images/RevoPAYTechnician/AN-MobilePayment_terminal.png)

### Once this conditions are satisfied, Technician's site selector will display your Site!

![ationetTR](Content/Images/RevoPAYTechnician/site_selector_helper.png)
