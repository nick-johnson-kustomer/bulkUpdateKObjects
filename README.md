# bulkUpdateKObjects
Workflow JSON code to bulk update KObjects, created by Nick Johnson

This build is specific to Calendly timeline objects, but it can be modified for any custom Klass. In this example, the client wanted the customer's Company Name to be populated in the Calendly objects. Before building this, I created a new custom attribute on the Calendly Klass. This set of workflows is designed to search for all Calendly objects that don't currently have a "Customer's Company" value, then isolate the first result from the search, then update that attribute with the Company Name.

This was built because searches do not currently allow bulk updates for KObjects. Now that Tasks is available, there may be more demand for bulk KObject updates.

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

Next we have 2 conditional branches. In this client's case, they have some customers who are not associated with a company. The left branch will proceed if a company was found. The right branch will proceed if a company was not found.

In either case, we move on to a step that updates the KObject to populate it with the Company Name.

Lastly, we have a scheduler step to loop this workflow around on itself so that it can proceed to update the next customer. There is a 1 minute delay in place because the search results do not update quickly enough after the KObject was updated, so an immediate re-run of the workflow will pull up the same search results several times until it eventually moves on to the next KObject. The 1 minute delay prevents unnecessary runs and helps to avoid rate limits. In a test case without a delay built in, the workflow cycled over 60 tiimes to update just 3 objects, so the delay is necessary especially for very large bulk updates.
