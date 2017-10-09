# solar-village-poc

Setup:

1. Start BPMS in standalone mode:
  - {bpms-install-location}/target/jboss-eap-6.4/bin/standalone.sh

2. Import project by navigating to Project Authoring -> Administration -> Clone Git Repository. Clone the following project:
  https://github.com/atef23/solar-village-poc.git
  
3. Add users and update roles. Run the following script for "salesUser" and "executiveUser":
  
  Supply the following roles for each user:
    salesUser=analyst,user,sales,kie-server
    executiveUser=analyst,user,executives,kie-server
    
  {bpms-install-location}/target/jboss-eap-6.4/bin/add-user.sh

4. Configure email for notifications:
  - run {bpms-install-location}/target/jboss-eap-6.4/bin/jboss-cli.sh -c --controller=127.0.0.1:9990
  - Configure an email of your choice to use as the smtp server. In this example, a gmail account is used. Run the following    CLI commands: 
  
  /socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=jbpm-mail-smtp/:add(host=smtp.gmail.com, port=465)
  
 /subsystem=mail/mail-session=jbpm/:add(jndi-name=java:/jbpmMailSession, from=username@gmail.com)  
 
 /subsystem=mail/mail-session=jbpm/server=smtp/:add(outbound-socket-binding-ref=jbpm-mail-smtp, ssl=true, username=username@gmail.com, password=password)     
  
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
