# solar-village-poc

Setup:

**1. Start BPMS in standalone mode:**
  - {bpms-install-location}/target/jboss-eap-6.4/bin/standalone.sh

**2. Import project by navigating to Project Authoring -> Administration -> Clone Git Repository. Clone the following project:**
  https://github.com/atef23/solar-village-poc.git

**3. Add users and update roles. Run the following script for "salesUser" and "executiveUser":**
Supply the following roles for each user:
      
        salesUser=analyst,user,sales,kie-server
        executiveUser=analyst,user,executives,kie-server
      
{bpms-install-location}/target/jboss-eap-6.4/bin/add-user.sh

**4. Configure email for notifications:**
  - run *{bpms-install-location}/target/jboss-eap-6.4/bin/jboss-cli.sh -c --controller=127.0.0.1:9990*
  - Configure an email of your choice to use as the smtp server. In this example, a gmail account is used. Run the following    CLI commands:
  
  */socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=jbpm-mail-smtp/:add(host=smtp.gmail.com, port=465)*
  
 */subsystem=mail/mail-session=jbpm/:add(jndi-name=java:/jbpmMailSession, from=username@gmail.com)*  
 
 */subsystem=mail/mail-session=jbpm/server=smtp/:add(outbound-socket-binding-ref=jbpm-mail-smtp, ssl=true, username=username@gmail.com, password=password)*     
  
(Please request pre-configured email from Atef if you prefer to use that)

  - Go to https://myaccount.google.com/security Click "Apps with account access" -> Allow less secure apps: ON
  
  - Navigate to {bpms-install-location}/target/jboss-eap-6.4/standalone/deployments/business-central.war/WEB-INF/classes
  - Add an email address to recieve escalation notification at. Add to userinfo.properties:
      
      executiveUser=username@emailserver.com:en-UK:executiveUser
      executives=:en-UK:executives:[executiveUser]
      
  - Log in to localhost:8080/business-central. Click Project Authoring -> Open Project Editor -> Project Settings Dropdown ->       Deployment Descriptor -> Under "work item handlers" click "Add" -> 

        name: Email
        identifier: new org.jbpm.process.workitem.email.EmailWorkItemHandler(null,"-1",null,null,true)
        resolver type: mvel
        
  - Save and restart server
  
  
**5. Deploy Project**
- Under Deploy -> Execution Servers go to your remote KIE server and click "Add Container"
- Select 1.2 version of com.redhat.poc:solarVillage:1.2 and start container

**6. Deploy Government Approvals REST Services**
- Download government approvals rest project (this will mock endpoints for the permit approval process): git clone https://github.com/atef23/solar-village-govt-approvals.git
- run "mvn clean install wildfly:deploy" in solar-village-govt-approvals directory

**7. Configure User Task for Email Test**
- The escalation on the user task for HOA Approvals occurs 10 days before the HOA meeting date if the task has not been claimed. To test this feature change the task expiration with the following steps:
- Navigate to the New Order Permitting business process in the business central workbench.
- Click on "HOA Approval" user task and change the ExpiresAt attribute from                              "#{hoaMeetingExpiration}" to "1m". This will escalate the task after 1m for email testing.

**Use the following CURL commands to run the process**

**Start process:**
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" --user bpmsAdmin:bpmsuite1! -d '{"orderID":"1231234","orderDate":"10/10/2017","hoaMembership":true,"hoaMeetingDate":"11/10/2017","propertyAddressLine1":"8205 Willow Dr.","city":"Dallas","state":"Texas","country":"USA","zipCode":23456}' http://localhost:8080/kie-server/services/rest/server/containers/solarvillagepoc/processes/solarVillage.NewOrderPermitting/instances

**Get process instances:**
curl -X GET -H "Accept: application/json" --user username:password http://localhost:8080/kie-server/services/rest/server/queries/containers/solarvillagepoc/process/instances

**Get potential owner sales group:**
curl -X GET -H "Accept: application/json" --user username:password "http://localhost:8080/kie-server/services/rest/server/queries/tasks/instances/pot-owners?groups=sales"

**Start task (Substitute taskID for "1"):**
curl -X PUT -H "Accept: application/json" --user salesUser:password "http://localhost:8080/kie-server/services/rest/server/containers/solarvillagepoc/tasks/1/states/started"

**Claim task by sales user (Substitute taskID for "1"):**
curl -X PUT -H "Accept: application/json" --user salesUser:password "http://localhost:8080/kie-server/services/rest/server/containers/solarvillagepoc/tasks/1/states/claimed"

**complete task (Substitute taskID for "1"):**
curl -X PUT -H "Accept: application/json" -H "Content-Type: application/json" --user salesUser:password -d '{"hoaApproval":true}' "http://localhost:8080/kie-server/services/rest/server/containers/solarvillagepoc/tasks/1/states/completed"
