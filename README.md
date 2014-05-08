## Create NACHA files in PHP

Supports debit (collecting money from others) and credit (transferring money to others) transactions.
If you need more functionality, I'm available for contract work at sparrish@nodeping.com

## Usage:

```php
include 'Nacha.php';
$nacha = new NachaFile();
$nacha
      /** Your bank's routing number */
      ->setBankRT('123456789')
      /**
       * '200' if expected to debit some accounts and credit others in this transaction
       * '220' if expected to only credit accounts
       * '225' if expected to only debit accounts
       */
      ->setServiceClassCode('200')
      /**
       * 'PPD' if consumer payments [default]
       * 'CCD' if company-to-company payments
       * 'CTX' if corporate trade
       * 'TEL' if telephone-initiated entries
       * 'WEB' if auth over the internet
       */
      ->setSECCode('CCD')
      /** Provided by bank (usually similar to your company Fed tax id) */
      ->setCompanyId('9876543210')
      /** Provided by bank (usually same as Company Id) */
      ->setFileID('9876543210')
      /** Text name of your bank */
      ->setOriginatingBank('BANK ON IT')
      /** File of the day [usually 'A'] ('A' for 1st, 'B' for 2nd, ..., 'Z' if 26th) */
      ->setFileModifier('A')
      /** Text name of your company (24 chars) */
      ->setCompanyName('MY COMPANY')
      /** Text description for batch (anything you like) */
      ->setBatchInfo('Monthly Subscriptions')
      /** Text description for batch (anything you like) and optional date */
      ->setDescription('Subscription', '05/17/2012')
      /** Internal reference code for file (8 chars) */
      ->setReferenceCode('20120526')
      /** Date in which to process this NACHA, usually the current date */
      ->setEntryDate(date('m/d/Y'));
```

Then push in the customer payments.
Each customer record should look like:

```php
$payment = array(
                 "AccountNumber"=>'1234567', // customer id your records
                 "TotalAmount" => 30.00, // Amount they are paying you if it's a debit - or that you're paying them if it's a credit.
                 "RoutingNumber"=>'987654321', // Customer's bank routing number
                 "BankAccountNumber"=>'123456789', // Customer's bank account number
                 "FormattedName" => 'Joe Smith', // Customer's name
                 "AccountType" => 'CHECKING' // 'CHECKING' or 'SAVINGS'
);

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
