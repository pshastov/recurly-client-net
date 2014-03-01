#Accounts

- [List](#list-accounts)
- [Get](#get-account)
- [Create](#create-account)
- [Close](#close-account)
- [Reopen](#reopen-account)
- [List Account Notes](#list-account-notes)

###List Accounts
```c#
var accounts = Accounts.List();
while (accounts.Any())
{
	foreach (var account in accounts)
		Console.WriteLine(account);
	accounts = accounts.Next;
}
```

###Get Account
```c#
try
{
	var account = Accounts.Get("1");
	Console.WriteLine("Account " + account);
}
catch (NotFoundException e)
{
	Console.WriteLine("Account not found.");
}
```
**Please note**: the client library will raise an exception if the account is not found.

###Create Account
```c#
var account = new Account("1")
{
	Email = "verena@example.com",
	FirstName = "Verena",
	LastName = "Example"
};
account.Create();
```

###Close Account
```c#
var account = Accounts.Get("1");
account.Close();
```

###Reopen Account
```c#
var account = Accounts.Get("1");
account.Reopen();
```

###List Account Notes
```c#
var account = Accounts.Get("1");
var notes = account.GetNotes();
while (notes.Any())
{
	foreach (var note in notes)
		Console.WriteLine("Note: " + note.Message);
	notes = notes.Next;
}
```

#Account Adjustments

- [List](#list-adjustments)
- [Get](#get-adjustment)
- [Create](#create-adjustment)
- [Delete](#delete-adjustment)

###List adjustments

```c#
var account = Accounts.Get("1");
var adjustments = account.GetAdjustments();
while (adjustments.Any())
{
	foreach (var adjustment in adjustments)
		Console.WriteLine("Adjustment: " + adjustment);
	adjustments = adjustments.Next;
}
```

###Get adjustment

```c#
var adjustment = Adjustments.Get("123456789");
Console.WriteLine("Adjustment: " + adjustment);
```

###Create adjustment

```c#
var account = Accounts.Get("1");
var adjustment = account.CreateAdjustment(
	"Charge for extra bandwidth", // description
	5000,                         // unit_amount_in_cents
	"USD",                        // currency
	1,                            // quantity
	"bandwidth",                  // accounting_code
	false);                       // tax_exempt
adjustment.Create();
```

###Delete adjustment

```c#
var adjustment = Adjustments.Get("123456789");
adjustment.Delete()
```

#BillingInfo

- [Get](#get-billing-info)
- [Update](#update-billing-info)
- [Delete](#delete-billing-info)

###Get Billing Info

```c#
var account = Accounts.Get("1");
var info = account.BillingInfo;
```

###Update billing info

```c#
var account = Accounts.Get("1");
var info = account.BillingInfo;
info.FirstName = "Verana";
info.LastName = "Example";
info.PhoneNumber = "111-111-1111";
info.VerificationValue = "123";
info.ExpirationMonth = 11;
info.ExpirationYear = 2015;
info.Update();
```

###Delete billing info

```c#
var account = Accounts.Get("1");
account.ClearBillingInfo(); // TODO rename to Delete()
```

#Coupons

- [List](#list-active-coupons)
- [Get](#get-coupon)
- [Create](#create-coupon)
- [Deactivate](#deactivate-coupon)

###List active coupons
```c#
var coupons = Coupons.List();
while (coupons.Any())
{
	foreach (var coupon in coupons)
		Console.WriteLine("Coupon: " + coupon);
	coupons = coupons.Next;
}
```

###Get Coupon
```c#
var coupon = Coupons.Get("special");
```

###Create coupon
```c#
// $2 off...
var coupon = new Coupon("special", "Special $2 off coupon", new Dictionary<string, int> {{"USD", 200}});

// ...or 10% off
var coupon = new Coupon("special", "Special $2 off coupon", 10);

// Other properties
coupon.RedeemByDate = new DateTime(2014, 1, 1);
coupon.SingleUse = true;

// Limit to gold and platinum plans only.
coupon.AppliesToAllPlans = false;
coupon.Plans.Add("gold");
coupon.Plans.Add("silver");

coupon.Create();
```

###Deactivate coupon
```c#
var coupon = Coupons.Get("special");
coupon.Deactivate();
```

#Coupon Redemptions

##Redeem a coupon before or after a subscription
```c#
var account = Accounts.Get("1");
var redemption = account.Redeem("special", "USD");
```

##Lookup a coupon redemption on an account
```c#
var account = Accounts.Get("1");
var redemption = account.GetActiveCoupon();
```

##Remove a coupon from an account
```c#
var account = Accounts.Get("1");
var redemption = account.GetActiveCoupon();
redemption.Delete();
```

##Lookup a coupon redemption on an invoice
```c#
var invoice = Invoices.Get(1);
var redemption = invoice.GetCoupon();
```

#Invoices

##List invoices
```c#
var invoices = Invoices.List();
while(invoices.Any())
{
	foreach(var invoice in invoices)
		Console.WriteLine("Invoice: " + invoice);
	invoices = invoices.Next;
}
```

##List an account's invoices
```c#
var invoices = Invoices.List("1"); // account code
while(invoices.Any())
{
	foreach(var invoice in invoices)
		Console.WriteLine("Invoice: " + invoice);
	invoices = invoices.Next;
}
```

##Lookup invoice details
```c#
var invoice = Invoices.Get(1005);
```

##Retrieve a PDF invoice
```c#
var invoice = Invoices.Get(1005);
byte[] pdf = invoice.GetPdf();
```

##Post an invoice: invoice pending charges on an account
```c#
var account = Accounts.Get("1");
var invoice = account.InvoicePendingCharges()
```

##Mark an invoice as paid successfully
```c#
var invoice = Invoices.Get(1005);
invoice.MarkSuccessful();
```

##Mark an invoice as failed collection
```c#
var invoice = Invoices.Get(1005);
invoice.MarkFailed();
```

##Line item refunds
```c#
var invoice = Invoices.Get(1005);
var adjustment = invoice.Adjustments.First(x => x.Uuid == "e1234245132465");
invoice = invoice.Refund(adjustment, false, 1); // adjustment, prorate, quantity
```

#Plans
##List plans
```c#
var plans = Plans.List();
while(plans.Any())
{
	foreach(var plan in plans)
		Console.WriteLine("Plan: " + plan);
	plans = plans.Next;
}
```

##Lookup plan details
```c#
var plan = Plans.Get("gold");
```

##Create plan
```c#
var plan = new Plan("gold", "Gold plan"); // plan code, name
plan.UnitAmountInCents.Add("USD", 1000);
plan.UnitAmountInCents.Add("EUR", 800);
plan.SetupFeeInCents.Add("USD", 6000);
plan.SetupFeeInCents.Add("EUR", 4500);
plan.PlanIntervalLength = 1;
plan.PlanIntervalUnit = Plan.IntervalUnit.Month;
plan.TaxExempt = false;
plan.Create();
```

##Update Plan
```c#
var plan = Plans.Get("gold");
plan.SetupFeeInCents["EUR"] = 5000;
plan.Update();
```

##Delete Plan
```c#
var plan = Plans.Get("gold");
plan.Deactivate();
```

#Plan Addons

##List add-ons for a plan
```c#
var plan = Plans.Get("gold");
var addons = plan.AddOns;
while(addons.Any())
{
	foreach(var addon in addons)
		Console.WriteLine("Addon: " + addon);
	addons = addons.Next;
}
```

##Lookup an add-on
```c#
var plan = Plans.Get("gold");
var addon = plan.GetAddOn("addOnCode");
```

##Create an add-on
```c#
var plan = Plans.Get("gold");
var addon = plan.CreateAddOn("ipaddresses", "Extra IP Addresses"); // add-on code, name
addon.UnitAmountInCents.Add("USD", 200);
addon.Create();
```

##Update an add-on
```c#
var plan = Plans.Get("gold");
var addon = plan.GetAddOn("ipaddresses");
addon.UnitAmountInCents["USD"] = 200;
addon.Update();
```

##Delete an add-on
```c#
var plan = Plans.Get("gold");
var addon = plan.GetAddOn("ipaddresses");
addon.Deactivate();
```

#Subscriptions

##List Subscriptions
```c#
var subscriptions = Subscriptions.List();
while(subscriptions.Any())
{
	foreach(var subscription in subscriptions)
		Console.WriteLine("Subscription: " + subscription);
	subscriptions = subscriptions.Next;
}
```

##List an account's subscriptions
```c#
var account = Accounts.Get("1");
var subscriptions = account.GetSubscriptions();
while(subscriptions.Any())
{
	foreach(var subscription in subscriptions)
		Console.WriteLine("Subscription: " + subscription);
	subscriptions = subscriptions.Next;
}
```

##Lookup subscription details
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
```

##Create Subscription
```c#
var account = Accounts.Get("1");
var plan = Plans.Get("gold");
var subscription = new Subscription(account, plan, "USD"); // account, plan, currency
subscription.Create();
```

##Update subscription
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.Plan = Plans.Get("silver");
subscription.Quantity = 2;
subscription.ChangeSubscription(Subscription.ChangeTimeframe.Now); // performs the update operation
```

##Cancel a subscription
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.Cancel();
```

##Reactivating a canceled subscription
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.Reactivate();
```

##Terminate a subscription
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.Terminate(Subscription.RefundType.Full);
//subscription.Terminate(Subscription.RefundType.Partial);
//subscription.Terminate(Subscription.RefundType.None);
```

##Postpone a subscription
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.Postpone(new DateTime(2012, 12, 31));
```

##Preview a subscription
```c#
var account = Accounts.Get("1");
var plan = Plans.Get("gold");
var subscription = new Subscription(account, plan, "USD"); // account, plan, currency
subscription.Preview();
```

#Subscription Add-ons

##Create a subscription with Add-Ons
```c#
var account = Accounts.Get("1");
var plan = Plans.Get("gold");
var subscription = new Subscription(account, plan, "USD"); // account, plan, currency
subscription.AddOns.Add(new SubscriptionAddOn("extra_users", 1000, 2));
subscription.AddOns.Add(new SubscriptionAddOn("extra_ips", 100, 3));
subscription.Create();
```

##Update subscription with add-ons
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");

// append a new add-on
var newAddOn = new SubscriptionAddOn("my_new_add_on", 100, 2);
subscription.AddOns.Add(newAddOn);

// change a quantity of an existing add-on
var existingAddOn = subscription.AddOns.First(x => x.AddOnCode == "extra_ips");
existingAddOn.Quantity = 6;

// remove an add-on
subscription.AddOns.RemoveAt(0);

// remove all add-ons
subscription.AddOns.Clear();

// call for an update
subscription.ChangeSubscription(Subscription.ChangeTimeframe.Now);
```

#Subscriptions for Manual Invoicing

##Create subscription (Manual Invoice)
```c#
var account = Accounts.Get("1");
var plan = Plans.Get("gold");
var subscription = new Subscription(account, plan, "USD"); // account, plan, currency
subscription.CollectionMethod = "manual";
subscription.NetTerms = 10;
subscription.PoNumber = "PO1234";
subscription.Create();
```

##Update subscription (Manual Invoice)
```c#
var subscription = Subscriptions.Get("44f83d7cba354d5b84812419f923ea96");
subscription.CollectionMethod = "manual";
subscription.NetTerms = 10;
subscription.PoNumber = "PO1234";
subscription.ChangeSubscription(Subscription.ChangeTimeframe.Now);
```

#Transaction

##List transactions
```c#
var transactions = Transactions.List();
while(transactions.Any())
{
	foreach(var transaction in transactions)
		Console.WriteLine("Transaction: " + transaction);
	transactions = transactions.Next;
}
```

##List an account's transactions
```c#
var account = Accounts.Get("1");
var transactions = account.GetTransactions();
while(transactions.Any())
{
	foreach(var transaction in transactions)
		Console.WriteLine("Transaction: " + transaction);
	transactions = transactions.Next;
}
```

##Lookup transaction
```c#
var transaction = Transactions.Get("a13acd8fe4294916b79aec87b7ea441f");
```

##Create Transaction
###Example with stored billing info
```c#
var transaction = new Transaction("1", 100, "USD"); // account code, unit amount in cents, currency
transaction.Create();
```

###Example with new billing info
```c#
var account = Accounts.Get("1");
account.BillingInfo = new BillingInfo(account.AccountCode)
{
	FirstName = "Verana",
	LastName = "Example",
	CreditCardNumber = "4111-1111-1111-1111",
	VerificationValue = "123",
	ExpirationYear = 2015,
	ExpirationMonth = 11
};
var transaction = new Transaction(account, 100, "USD");
transaction.Create();
```

##Refund transactions

###Partial Refund Example
```c#
var transaction = Transactions.Get("a13acd8fe4294916b79aec87b7ea441f");
transaction.Refund(1000); // Refund $10
```

###Full Refund Example
```c#
var transaction = Transactions.Get("a13acd8fe4294916b79aec87b7ea441f");
transaction.Refund();
```