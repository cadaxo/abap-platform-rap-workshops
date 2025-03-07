[Home - RAP100](../../#exercises)

# Exercise 5: Enhance the BO Behavior – Validations 

## Introduction 

In the previous exercise, you've defined and implemented a determination for setting the inital value of the field `OverallStatus` to `Open` (`O`) during the creation of new instances of BO entity _Travel_ (see [Exercise 4](../ex4/readme.md)).

In the present exercise, you're going to define and implement two validations, `validateCustomer` and `validateDates`, to respectively check if the customer ID that is entered by the consumer is valid and if the begin date is in the future and if the value of the end date is after the begin date.

- [5.1 - Define the Validation `validateCustomer` and `validateDates`](#exercise-51-define-the-validations-validatecustomer-and-validatedates)
- [5.2 - Implement the Validation `validateCustomer`](#exercise-52-implement-the-validation-validatecustomer)
- [5.3 - Implement the Validation `validateDates`](#exercise-53-implement-the-validation-validatedates)
- [5.4 - Preview and Test the enhanced Travel App](#exercise-54-preview-and-test-the-enhanced-travel-app)
- [Summary](#summary)  
- [Appendix](#appendix)

> **Reminder**: Do not forget to replace the suffix placeholder **`###`** with your choosen or assigned group ID in the exercise steps below. 

### Excursus: Validations

A validation is an optional part of the business object behavior that checks the consistency of business object instances based on trigger conditions. 

A validation is implicitly invoked by the business object’s framework if the trigger condition of the validation is fulfilled. Trigger conditions can be `MODIFY` operations and modified fields. The trigger condition is evaluated at the trigger time, a predefined point during the BO runtime. An invoked validation can reject inconsistent instance data from being saved by passing the keys of failed instances to the corresponding table in the `FAILED` structure. Additionally, a validation can return messages to the consumer by passing them to the corresponding table in the `REPORTED` structure.

> **Further reading**: [Validations](https://help.sap.com/viewer/923180ddb98240829d935862025004d6/Cloud/en-US/171e26c36cca42699976887b4c8a83bf.html)

## Exercise 5.1: Define the Validations `validateCustomer` and `validateDates`
[^Top of page](#)

> Define the validations `validateCustomer` and `validateDates`.  

 <details>
  <summary>Click to expand!</summary>
  
1. Open your behavior definition ![behaviordefinition](images/adt_bdef.png) **`ZRAP100_R_TRAVELTP_###`** to define the validations **`validateCustomer`** and **`validateDates`**.
     
   For that, add the following code snippet after the determination as shown on the screenshot below.
   
   ```ABAP
   validation validateCustomer on save { create; field CustomerID; }
   validation validateDates on save { create; field BeginDate, EndDate; }
   ```         

   In order to have draft instances being checked and determinations being executed before they become active, they have to be specified for the **`draft determine action prepare`** in the behavior definition.
  
   Replace the code line **`draft determine action Prepare;`** with the following code snippet as shown on the screenshot below

   ```ABAP
   draft determine action Prepare
   {
   validation validateCustomer;
   validation validateDates;    }
   ```    
     
   Your source code should look like this: 
   
   ![validation](images/new18.png)           
     
     
   **Short explanation**:    
   - Validations are always invoked during the save and specified with the keyword `validateCustomer on save`.    
   - `validateCustomer` is a validation with trigger operation `create` and trigger field `CustomerID`.    
   - `validateDates` is a validation with trigger operation `create` and trigger fields `BeginDate` and `EndDate`.            
    
2. Save ![save icon](images/adt_save.png) and activate ![activate icon](images/adt_activate.png) the changes.
      
3. Add the appropriate **`FOR VALIDATE ON SAVE`** methods to the local handler class of the behavior pool of the _Travel_ BO entity via quick fix.  
   
   For that, set the cursor on one of the validation names and press **Ctrl+1** to open the **Quick Assist** view and select the entry _**`Add all 2 missing methods of entity zrap100_i_travel_### ...`**_.
   
   As a result, the **`FOR VALIDATE ON SAVE`** methods **`validateCustomer`** and **`validateDates`** will be added to the local handler class `lcl_handler` of the behavior pool of the _Travel_ BO entity ![class icon](images/adt_class.png)`ZRAP100_BP_TRAVELTP_###`.       
   
   ![Travel BO Behavior Pool](images/new19.png)  

4. Save ![save icon](images/adt_save.png) and activate ![activate icon](images/adt_activate.png) the changes.

</details>
  
## Exercise 5.2: Implement the Validation `validateCustomer` 
[^Top of page](#)

> Implement the validation `validateCustomer` which checks if the customer ID (`CustomerID`) that is entered by the consumer is valid.   
> An appropriate message should be raised and displayed on the UI for each invalid value.

 <details>
  <summary>Click to expand!</summary>

1. First, check the interface of the new methods in the declaration part of the local handler class `lcl_handler` of the behavior pool of the _Travel_ BO entity ![class icon](images/adt_class.png)**`ZRAP100_BP_TRAVEL_###`**. 

   For that, set the cursor on the method name, **`validateCustomer`**, press **F2** to open the **ABAP Element Info** view, and examine the full method interface. 
   
   ![Travel BO Behavior Pool](images/v3.png)  
   
   **Short explanation**:  
   - The addition **`FOR VALIDATE ON SAVE`** indicates that the method provides the implementation of a validation executed on save. Validations are always executed on save.   
   -  Method signature for the validation method:
     - `IMPORTING`parameter **`keys`** - an internal table containing the keys of the instances on which the validation should be performed.
     - Implicit `CHANGING` parameters (aka _implicit response parameters_):  
       - **`failed`**   - table with information for identifying the data set where an error occurred
       - **`reported`** - table with data for instance-specific messages
      
   You can go ahead and implement the validation method.

2. Now implement the method **`validateCustomer`** in the implementation part of the class.
   
    The logic consists of the following main steps:   
    1. Read the travel instance(s) of the transferred keys (**`keys`**) using the EML statement **`READ ENTITIES`**. Only the field `CustomerID` is read in the present scenario.
    2. Read all the transfered (distinct, non-initial) customer IDs and check if they exist.  
    3. Prepare/raise messages for all transferred _travel_ instances with initial and non-existing customer ID (**`CustomerID`**)  
       and set the changing parameter **`reported`**   

   Replace the current method implementation of **`validateCustomer`** with following code snippet and replace all occurrences of the placeholder **`###`** with your group ID. 
   
   You can use the **F1 Help** to get detailed information on the different ABAP and EML statements.  
   
      ```ABAP
      **********************************************************************
      * Validation: Check the validity of the entered customer data
      **********************************************************************
      METHOD validateCustomer.
        " Read relevant travel instance data
        READ ENTITIES OF ZRAP100_R_TravelTP_### IN LOCAL MODE
        ENTITY Travel
         FIELDS ( CustomerID )
         WITH CORRESPONDING #( keys )
        RESULT DATA(lt_travel).

        DATA lt_customer TYPE SORTED TABLE OF /dmo/customer WITH UNIQUE KEY customer_id.

        " Optimization of DB select: extract distinct non-initial customer IDs
        lt_customer = CORRESPONDING #( lt_travel DISCARDING DUPLICATES MAPPING customer_id = customerID EXCEPT * ).
        DELETE lt_customer WHERE customer_id IS INITIAL.
        IF lt_customer IS NOT INITIAL.

          " Check if customer ID exists
          SELECT FROM /dmo/customer FIELDS customer_id
            FOR ALL ENTRIES IN @lt_customer
            WHERE customer_id = @lt_customer-customer_id
            INTO TABLE @DATA(lt_customer_db).
        ENDIF.
        " Raise msg for non existing and initial customer id
        LOOP AT lt_travel INTO DATA(ls_travel).
          IF ls_travel-CustomerID IS INITIAL
             OR NOT line_exists( lt_customer_db[ customer_id = ls_travel-CustomerID ] ).

            APPEND VALUE #(  TravelID = ls_travel-TravelID ) TO failed-travel.
            APPEND VALUE #(  TravelID = ls_travel-TravelID
                             %msg = new_message( id        = '/DMO/CM_FLIGHT_LEGAC'
                                                 number    = '002'
                                                 v1        = ls_travel-CustomerID
                                                 severity  = if_abap_behv_message=>severity-error )
                             %element-CustomerID = if_abap_behv=>mk-on )
              TO reported-travel.
          ENDIF.
        ENDLOOP.
      ENDMETHOD.   
      ```   
  
3. Save ![save icon](images/adt_save.png) and activate ![activate icon](images/adt_activate.png) the changes.

</details>

## Exercise 5.3: Implement the Validation `validateDates` 
[^Top of page](#)

> Implement the validation `validateDates` which checks the validity of entered begin dates (`BeginDate`) and end dates (`EndDate`). 
> An appropriate messages should be raised and displayed on the UI for each invalid value.

 <details>
  <summary>Click to expand!</summary>

1. In your implementation class ![class](images/adt_class.png) **`ZRAP100_BP_TRAVELTP_###`**, replace the current method implementation of **`validateDates`** with following code snippet and replace all occurrences of the placeholder **`###`** with your group ID.

   The main implementation steps are similar to the one of method **`validateCustomer`**. The only main difference is the validation logic. 
   
   This validation checks if the entered begin date (`BeginDate`) is in the future and if the value of the entered end date (`EndDate`) is after the begin date (`BeginDate`).       
   ```ABAP
   **********************************************************************
   * Validation: Check the validity of begin and end dates
   **********************************************************************
     METHOD validateDates.
       READ ENTITIES OF ZRAP100_R_TravelTP_### IN LOCAL MODE
         ENTITY travel
           FIELDS ( BeginDate EndDate )
           WITH CORRESPONDING #( keys )
         RESULT DATA(lt_travel_result).

       LOOP AT lt_travel_result INTO DATA(ls_travel_result).

         IF ls_travel_result-EndDate < ls_travel_result-BeginDate.  "if end_date is before begin_date

           APPEND VALUE #( %key        = ls_travel_result-%key
                           travelID    = ls_travel_result-TravelID ) TO failed-travel.

           APPEND VALUE #( %key = ls_travel_result-%key
                           %msg     = new_message( id       = /dmo/cx_flight_legacy=>end_date_before_begin_date-msgid
                                                   number   = /dmo/cx_flight_legacy=>end_date_before_begin_date-msgno
                                                   v1       = ls_travel_result-BeginDate
                                                   v2       = ls_travel_result-EndDate
                                                   v3       = ls_travel_result-TravelID
                                                   severity = if_abap_behv_message=>severity-error )
                           %element-BeginDate = if_abap_behv=>mk-on
                           %element-EndDate   = if_abap_behv=>mk-on ) TO reported-travel.

         ELSEIF ls_travel_result-BeginDate < cl_abap_context_info=>get_system_date( ).  "begin_date must be in the future

           APPEND VALUE #( %key       = ls_travel_result-%key
                           travelID   = ls_travel_result-TravelID ) TO failed-travel.

           APPEND VALUE #( %key = ls_travel_result-%key
                           %msg = new_message( id       = /dmo/cx_flight_legacy=>begin_date_before_system_date-msgid
                                               number   = /dmo/cx_flight_legacy=>begin_date_before_system_date-msgno
                                               severity = if_abap_behv_message=>severity-error )
                           %element-BeginDate = if_abap_behv=>mk-on
                           %element-EndDate   = if_abap_behv=>mk-on ) TO reported-travel.
         ENDIF.
       ENDLOOP.
     ENDMETHOD.
   ```  

2. Save ![save icon](images/adt_save.png) and activate ![activate icon](images/adt_activate.png) the changes.

</details>

## Exercise 5.4: Preview and Test the enhanced Travel App
[^Top of page](#)

Now the SAP Fiori elements app can be tested. 

 <details>
  <summary>Click to expand!</summary>

You can either refresh your application in the browser using **F5** if the browser is still open - or go to your service binding **`ZRAP100_UI_TRAVEL_O4_###`** and start the Fiori elements App preview for the **`Travel`** entity set.

1. Click **Create** to create a new entry.

2. Select an `Sunshine Travel (70001)` as Agency ID,  **12345** as Customer ID, Mar 16, 2022 as starting date and Mar 14, 2022 as end date and click **Create**.

     ![package](images/preview3.png)
     
3. You should get following message displayed.

    ![package](images/preview4.png)
    
</details>

## Summary 
[^Top of page](#)

Now that you've... 
- defined two validations in the behavior definition, 
- implemented them in the behavior pool, and
- preview and test the enhanced Fiori elements app,

you can continue with the next exercise – **[Exercise 6: Enhance the BO Behavior – Actions](../ex6/readme.md)**

---

## Appendix
[^Top of page](#)

Find the source code for the behavior definition and behavior implementation class (aka behavior pool) in the [sources](sources) folder. Don't forget to replace all occurences of the placeholder `###` with your group ID.

- ![document](images/doc.png) [CDS BDEF ZRAP100_R_TRAVELTP_###](sources/EX5_BDEF_ZRAP100_R_TRAVELTP.txt)
- ![document](images/doc.png) [Class ZRAP100_BP_TRAVELTP_###](sources/EX5_CLASS_ZRAP100_BP_TRAVELTP.txt)

