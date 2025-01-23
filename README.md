# LucaP_ERP-03
# LucaPERP part 3: The General Journal 
By Steven Lipton
#LucaP


In #LucaP, we're building an ERP system starting at the core: the General Ledger. This week, we'll tie together what we've done by building the General journal, discuss how to work with one-to-many relationships and build a report with headers and footers. #ERP #Accounting #SAPB1 #Bizoneness #Swift #SwiftUI


**Note:As I've gained a few followers and subscriptions in the last few weeks, I want to remind everyone that I alternate between LucaP and BizOneness weekly.*

In LucaP, we're building an ERP system starting at the core: the General Ledger. This week, we'll tie together what we've done by building the General journal. 

I'm going to change the terms I've used so far. Entries in the general journal will be *journal entries*. The credit and debit entries we've used will become *journal transactions*. The journal entry gives us the story of a set of transactions.

Suppose  we had these two transactions: 


It is hard to gain enough information from this. Here's a journal entry for those transactions: 

We can quickly learn everything about this entry. Huli Pizza purchased $100.00 of chicken from Mike's, their chicken vendor, and will pay them later. 

We can make more than two transactions in an entry, for example, this revenue entry after selling pizzas in a tent at a flea market:

Huli Pizza took in $5000 cash as revenue, but we also decreased inventory by $400 as the costs of goods sold, requiring four transactions.  

We'll organize all our entries like this, then write reports off those entries. But first, we need to structure this data. 

Data dependent on other data is called a *parent-child relationship*. 

When we worked with the chart of accounts, we fetched the chart of account data from within the transaction using the account number. There was one account to one account number. we refer this as *one-to-one relationship*, with one parent and one child. 

However, with the journal entry, we get one entry for multiple transactions. This is a *one-to-many relationship*, with one parent and multiple children. Developers look at this relationship differently than one-to-one relationships. 

One-to-many relationships commonly present themselves in a similar format to what we have here: a header/body/footer format. At the top is the header with information about the entry. In the middle is a report of rows that are the multiple transactions. On the bottom is the footer, which contains less relevant information from the parent or summarizes the rows. You'll see this pattern in A/R and A/P invoices, sales orders, Quotes, and Production Orders, among other documents. 

For now, we want to build the data structure. Common information for the entry would be the date and description of the transaction. The totals for all the transactions are helpful as well. As an option, a remarks column that has more information about the transaction can be handy. Finally, check if the credits and debits are in balance. 

Of course, you'll also have a collection of transactions, and the one-to-many relationship presents the most significant challenge here. For a one-to-one, you could store the data in a row, as we did with the description of the GL Accounts. However, there are several reasons not to do this. Unlike the accounts, we want to reflect changes in the transactions, though there shouldn't be any. We also want to summarize and report on transactions quickly. A table of only transactions, like the one we've already made, does so efficiently. Finally, conserving data space is essential here, as this will take up more space, especially if we try to use both a table of transactions and transactions in the entry. Therefore, the best way is to have a reference for the transactions. We mark the transactions with a key value, the entry's unique identifier, to link it as belonging to the entry. 

The entries I had before as 2 and 3

are now both 102

Initially, I marked them 2 and 3 because they were unique row identifiers, and they still need to be. So I'll make another key for the row, marking them as 0 and 1:

Having those rows available is good for reporting and later edits. 

I make one more key for internal indication of uniqueness. Databases, or in my case, the SwiftUI `Identifiable` protocol, require keys, often combining into a single value such as `10200` and `10201` as an integer or `102-00` and `102-01` as a string. In general, I like integers for this internal ID, as it is one of the smallest data forms, and I try to be conservative with my data space, plus it tends to to be quickest in searches. However, there are cases where string works better, especially if user-facing. 

Our schema of all the tables we've made so far looks like this: 

I changed the table slightly to add notations for one to many. 

With that, we are ready to code. I'll make a few changes to the code to fix our nomenclature. In developer speak, this is known as *refactoring*. I'll refactor all the `journalEntry` stuff to `journalTransaction`. 

Our first task is changing the coding on the transactions to the `docId` and `row` structure. 

```
   // Id becomes two keys 
    var docId:Int
    var row:Int
```

And I'll change the `init` to reflect this: 

``` init(docId:Int,row:Int,account:String,amount:Double,creditDebit:CreditDebit){
        //id is no longer automatically assigned, replaced with docId and row
       // id = globalJournalEntryID
       // globalJournalEntryID += 1
        self.docId = docId
        self.row = row
        ...
```

This code change removes `id`. `id` is our internal primary key and still required by `Identifiable`. There are several solutions to generating this key. One strategy is to generate it in sequence like the `id`'s we've done already. I've combined the entry and row into a single `id` as a text or a number. I prefer integers for both speed and size. Assuming there will be less than 100 transactions on any given entry, I'll create an `id` by multiplying the `docId` by 100 and then adding the row. 

```
var id:Int{
   docId * 100 + row
}
```

The last part is to set up my testing. I made a new class to set up the test transactions like this, changing the data  I need for these new changes: 

```
class JournalTransactions{
    
    var transactions:[JournalTransaction] 
    
    init(){
        self.transactions = testTransactions 
    }
    
    //changed name to testTransactions
    var testTransactions = [
        JournalTransaction(docId:100,row: 0, account:"321000", amount: 1000, creditDebit: .credit),
        JournalTransaction(docId:100,row: 1,account:"111000", amount: 1000, creditDebit: .debit),
        JournalTransaction(docId:101,row: 0,account:"131100", amount: 300, creditDebit: .debit),
        JournalTransaction(docId:101,row: 1,account:"111000", amount: 300, creditDebit: .credit),
        JournalTransaction(docId:102,row: 0,account:"131100", amount: 100, creditDebit: .debit),
        JournalTransaction(docId:102,row: 1,account:"211000", amount: 100, creditDebit: .credit),
        JournalTransaction(docId:103,row: 0,account:"621000", amount: 300, creditDebit: .debit),
        JournalTransaction(docId:103,row: 1,account:"111000", amount: 300, creditDebit: .credit),
        JournalTransaction(docId:104,row: 0,account:"652100", amount: 200, creditDebit: .debit),
        JournalTransaction(docId:104,row: 1,account:"111000", amount: 200, creditDebit: .credit),
        JournalTransaction(docId:105,row: 0,account:"111000", amount: 5000, creditDebit: .debit),
        JournalTransaction(docId:105,row: 1,account:"411000", amount: 5000, creditDebit: .credit),
        JournalTransaction(docId:105,row: 2,account:"511000", amount: 400, creditDebit: .debit),
        JournalTransaction(docId:105,row:3,account: "111000",amount: 400,creditDebit: .credit)
    ]
}
```

We're ready to do the journal entry. I'll make a new file for the `JournalEntry` class and start with the simple columns: 

```
class JournalEntry:Identifiable{
var docId:Int
var date:Date()
var description:String
var remarks:String = ""
var id:Int{docId}
```

To initialize these, I'll use the description only. I will automatically add the `docID` the way I had been doing it with transactions before this, then default the date to the current date. I also added a remarks column, which I'm not using yet, but will come in handy later. When other modules, such as Accounts Receivable, start working, `description` gets automatic descriptions from those modules. For any other comments about the transaction, I'll use `remarks**. 
 
```
init(description:String){
		self.docId = globalJournalEntryID
    globalJournalEntryID += 1
    self.date = Date()
    self.description = description
}
```

For testing, we'll load entries with *docId*'s, so I'll make another initializer for that case:

```
 init(docId:Int,description:String, date:Date = Date()){
        self.docId = docId
         if globalJournalEntryID < docId{
             globalJournalEntryID = docId
         }
        self.date = Date()
        self.description = description
    }
```

Notice two changes from the first initializer. The `docId` is assigned here. I still want it unique, and `globalEntryID` will give me the next number. For now, I'm taking a strategy. If the docID is greater than the Global entry, reset the global entry. 

We're missing journal transactions. Those are on a different table, though I might need to access them. instead of adding them to the entry, I fetch them from the other table when I need them:

```
/// Filter to only transactions with this `docId`
/// - In pseudo SQL `SELECT t1.* FROM t0 INNER JOIN t1 ON t1.docId = t0.docId`
func fetchJournalTransactions()->[JournalTransaction]{
    let transactionTable = JournalTransactions().transactions
    return transactionTable.filter({$0.docId == docId})
}
```

In context, this is not a column but a joined table. For clarity in my code, I keep it a function and not a computed variable like I did with `id`. 

That gets us our header. The final part is the footer. I want to have sums of the journal transactions and a data check to see if this entry has balanced transactions. 

With the exception of joined tables, I want as many forward-facing columns as possible and try to avoid functions. If I need a function, I'll wrap them in properties. 

The total amount of credits and debits is a good example of this. I'll make one function to compute the sum of either a credit or a debit. 

```
 /// Compute the total credits or debits in the transactions
    func totalAmount(creditDebit:CreditDebit)->Double{
        var total:Double = 0
        for row in fetchJournalTransactions(){
            if row.creditDebit == creditDebit{
                total += row.amount
            }
        }
        return total
    }
```

Then I'll use this function to create computed properties: 

```
var totalDebits:Double{totalAmount(creditDebit: .debit)}
var totalCredits:Double{totalAmount(creditDebit: .credit)}
```

I'll use both for a simple data check. To paraphrase Luca Pacioli, "Dont go to sleep until credits equal debits." So that's our data check: 

```
var balancedEntry:Bool{totalDebits == totalCredits}
```

Finally, we add a class for sample input. I'll use this class until I add the persistence system, which will be SwiftData for this project. When working in computer languages as a base instead of a database, I test everything to work with arrays before I try to get it working with persistent storage. This way, I segregate the persistent storage bugs from all other bugs. I fix a lot of issues faster, and I have fewer data migrations to worry about. 

```
class JournalEntries{
    var journalEntries:[JournalEntry]
    
    init(){
        self.journalEntries = testEntries
    }
    
    var testEntries = [
        JournalEntry(docId: 100, description: "Initial investment"),
        JournalEntry(docId:101, description: "Purchase Ingredients - CostCo"),
        JournalEntry(docId: 102, description: "Purchase Ingredients - Mike's Chicken"),
        JournalEntry(docId: 103, description: "Rent Pizza Oven - Oahu Kitchen Rental"),
        JournalEntry(docId: 104, description: "Rent Sales Space - Swap Meet"),
        JournalEntry(docId: 105, description: "Sales - Swap Meet")
    ]
}

```

You'll notice the phrasing I gave the descriptions. These will eventually be automatically generated. When vendors or sales are generated, I include the name of the vendor to help track the purchases and sales. 


We've come to our testing report. I'm rewriting the test report to handle these changes, including a header, footer, and body. 

I set our two tables as arrays in this view:

```
struct EntryTestView: View {
// Set for test entries
    @State var journalEntries:[JournalEntry] = JournalEntries().journalEntries

```


Using the Section view, I can break up my report into entries and transactions. Inside the section goes the transactions. I fetch the transaction for only that entry. 

```
var body: some View {
    List{
        ForEach(journalEntries){entry in
            Section { 
                ForEach(entry.fetchJournalTransactions()){transaction in
                
                        } 
                        .padding([.leading,.trailing],10)
                    }
                } 
```

I'll list transactions by row number. Rows run into a problem between developers and users. Developers start counting from zero, so there is a zero column, though users tend to start at one. For user ease, I went with the first row as one here but with a cost: Any queries on this data I need to remember that I start at zero, not one. 

I'll put the account, description, and amount like I did before for the rest of the row. 

```
HStack{
	HStack{
    Text(transaction.row + 1,format: .number)
    Text(transaction.account)
		Text(transaction.description)
    Spacer()
	}
  .frame(width:300)
  Spacer()
  if transaction.creditDebit == .debit{
    HStack{
	    Text(transaction.amount,format: .currency(code: "USD"))
	    Spacer()
    }.frame(width:200)
	} else {
    Text(" ").frame(minWidth:100)
    Text(transaction.amount,format: .currency(code: "USD")).frame(width:100)
}
```

Due to the structure of `Section`, once I finish the body, I add the header with the entry data. Here, I added the `docId`, the formatted date, and the description, 

```
header: { 
	HStack{
    HStack{
        Text(entry.docId,format: .number.grouping(.never))
        Text(entry.date,format: .dateTime.month(.twoDigits).day(.twoDigits).year(.twoDigits))
        Text(entry.description)
        Spacer()              
    }
    .font(.headline)
    .foregroundColor(.primary)           
	}
}
```

Finally, under the header I add the footer. I use my `balancedEntry` property to check for errors. First, I put an error message up
                 
```
footer: {
	HStack{
    if !entry.balancedEntry{
	    Text("Not Balanced")
		    .padding(3)
        .padding(.trailing,100)
        .foregroundStyle(.white)
        .background(.red,in:RoundedRectangle(cornerRadius: 6))                        
		}
```

I'll add the total credits and debits. I'll also change the color to orange if they are not balanced. 


```
	Spacer()
	Text(entry.totalDebits,format: .currency(code: "usd"))
		.frame(width: 100)
	Text(entry.totalCredits,format: .currency(code: "usd"))
		.frame(width: 100)
}
	.foregroundColor(entry.balancedEntry ? .primary : .orange)
}  
```


And with that, we are ready to test. I'll run this and get the following transaction entry list: 

If I were to comment out the last line of the transactions: 

```
// JournalTransaction(docId:105,row:3,account: "111000",amount: 400,creditDebit: .credit)
```

I get my error in balancing showing this:

We've now built the core part of the general ledger, tying transactions to events we record as a journal entry. Everything else we do will tie back to this, either as reporting or adding information. We showed an example of a one-to-many relationship, fetching the child data when needed. 
However, we've only built the model and added some test data directly. Our next steps are designing UI to add, change and delete data, then to build reports that make sense of this data. 

In two weeks, we'll begin the form-building process, exploring the two fundamental types of forms you see in ERP systems. Next week, it's back to BizOneness for a survey of HANA functions and types. 
