# bulkUpdateKObjects
Workflow JSON code to bulk update KObjects

This build is specific to Calendly timeline objects, but it can be modified for any custom Klass. In this example, the client wanted the customer's Company Name to be populated in the Calendly objects. Before building this, I created a new custom attribute on the Calendly Klass. This set of workflows is designed to search for all Calendly objects that don't currently have a "Customer's Company" value, then isolate the first result from the search, then update that attribute with the Company Name.

This process consists of 2 workflows:

# Workflow 1

The first workflow is just a trigger. The 2nd workflow is callable, so this workflow just exists so that you can easily call the 2nd workflow, which will then loop around on itself.

This workflow requires an email hook, so you'll need to create a new email hook in your org and set it as the trigger.

Then make sure that the scheduler step calls the 2nd workflow. Set the timing to 0 minutes to get it to fire immediately. The input parameters in the optional fields can be anything, that parameter is not actually used in the 2nd workflow.

# Workflow 2

This workflow is responsible for the updates.

Step 1 is the callable trigger.

Step 2 is an API step calling the search endpoint. It is specifically looking for Calendly objects which don't have a "Customer's Company" value. You will need an appropriate auth token in the workflow variables for this to work.

Step 3 is a regex step designed to find the first Calendly object ID within the output of step 2.

The conditional step checks to see if an ID was found before proceeding. This step was included as an automatic off-switch. Once all of the Calendly objects have been updated, this conditional step will stop the workflow from looping on itself infinitely.

Step 4 takes the Calendly ID and looks up the Calendly object. It does this so we can find the customer ID.

Step 5 takes the customer ID and looks up the Customer data. It does this so we can find the Company ID.

Step 6 takes the company ID and looks up the company data. It does this so we can find the Company Name.

