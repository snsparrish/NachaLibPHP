## Create NACHA files in PHP

Supports debit (collecting money from others) and credit (transferring money to others) transactions.
If you need more functionality, I'm available for contract work at sparrish@nodeping.com

## Usage:

```php
include 'Nacha.php';
$nacha = new NachaFile();
$nacha->setBankRT('123456789')// Your bank's routing number
      ->setCompanyId('9876543210')// Usually your company Fed tax id with something the bank tells you to put on the front.
      ->setSettlementAccount('44444444444') // Your bank account you want the money to go into
      ->setFileID('9876543210')// Probably the same as your Company ID but your bank will tell you what this should be.
      ->setOriginatingBank('BANK ON IT')// Text name of your bank
      ->setFileModifier('A')// Usually just A - for the first file of the day.  Change to 'B' for second file of the day and so on.
      ->setCompanyName('MY COMPANY')//16 chars - your company name
      ->setBatchInfo('Monthly Subscriptions') // Text description for the batch
      ->setDescription('Subscription', '05/17/2012') // Description shown on customers statements and date of invoice
      ->setEntryDate(date('m/d/Y')); // The day you want the payments to be processed. This example shows today.
```

Then push in the customer payments.
Each customer record should look like:

```php
$payment = array("AccountNumber"=>'1234567', // The customer's CRM account number (not bank account number)
                 "TotalAmount"=>30.00, // Amount they are paying you if it's a debit - or that you're paying them if it's a credit.
                 "BankAccountNumber"=>'123456789', // Customer's bank account number
                 "RoutingNumber"=>'987654321', // Customer's bank routing number
                 "FormattedName"=>'Joe Smith', // Customer's name
                 "AccountType"=> 'CHECKING'); // Could be 'CHECKING' or 'SAVINGS' - customer's bank account type

// Add a debit - taking money from someone and puting it in your account
if(!$nacha->addDebit($payment)){
   // Error adding this debit.  Must be something wrong.  Check the $nacha->errorRecords.
}

// You can safely mix debits and credits in the same batch.
// Add a credit - sending your money to someone elses bank account
if(!$nacha->addCredit($payment)){
   // Error adding this credit.  Must be something wrong.  Check the $nacha->errorRecords.
}

// Generate the NACHA file contents
try{
    $nacha->generateFile();
    // Put the file contents on the file system
    if(!file_put_contents('./ACH_MyCompany_NACHA_file.txt', $nacha->fileContents)){
        throw new Exception('Unable to save NACHA file');
    }
}catch(Exception $e){
    // Something went wrong with the file generation
    print_r($e->getMessage());
}
```

Copyright 2012 sparrish@nodeping.com
