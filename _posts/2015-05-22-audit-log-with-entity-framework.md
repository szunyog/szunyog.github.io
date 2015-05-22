---
layout: post
title:  "Audit log with entity framework"
date:   2015-05-22 20:31:49
categories: ["c#", "EntityFramework", ".net"]
---

The EntytyFramework's context has the list of the created, changed, deleted entities, so it
is not so hard to create an audit log using these values.

First of all, the SaveChanges method of the DBContext should be overridden.
I created two event, one for before and one for after saving.
The reason why need we two is because the database generated keys,
such as auto-increment id, have values only after saving.

{% highlight csharp %}

public delegate void SavingEventHandler(DataContext context);
public event SavingEventHandler BeforeSaveChanges;
public event SavingEventHandler AfterSaveChanges;

public System.Data.Entity.Core.Objects.ObjectStateManager ObjectStateManager
{
    get
    {
        return ((IObjectContextAdapter)this).ObjectContext.ObjectStateManager;
    }
}
public override int SaveChanges()
{
    using (var tran = TransactionWrapper.BeginTransaction())
    {
        this.ChangeTracker.DetectChanges();

        if (this.BeforeSaveChanges != null)
            this.BeforeSaveChanges(this);

        var returnValue = base.SaveChanges();

        if (this.AfterSaveChanges != null)
            this.AfterSaveChanges(this);

        tran.Commit();
        return returnValue;
    }
}


{% endhighlight %}

The second important thing is create a factory class which is initializes the context
and attaches its Before and AfterSavingChanges events.

{% highlight csharp %}

public static DataContext Create()
{
    try
    {
        EnsureDatabaseInitialized();
        var context = new DataContext();
        var autditExtension = new Audit.AuditHandler();
        context.BeforeSaveChanges += autditExtension.BeforeSaveChanges;
        context.AfterSaveChanges += autditExtension.AfterSaveChanges;
        return context;
    }
    catch (Exception ex)
    {
		... long excpetion handling ...
	}
}
{% endhighlight %}

Before the SaveChanges the Create and Modify types should be collected to a member list,
the values of these entries will be fixed after save complete.

The Delete entries are added directly to the context because entity information will be lost after saving.

{% highlight csharp %}

public void BeforeSaveChanges(DataContext context)
{
    List<ObjectStateEntry> objectStateEntryList =
        context.ObjectStateManager.GetObjectStateEntries(EntityState.Added | EntityState.Modified | EntityState.Deleted).ToList();

    this.AuditList = new Dictionary<AuditLog, ObjectStateEntry>();
    foreach (ObjectStateEntry entry in objectStateEntryList)
    {
        if (ExcludedTypes.Contains(entry.Entity.GetType()))
            continue;

        if (!entry.IsRelationship)
        {
            if (entry.State == EntityState.Modified)
            {
                // The original values does not always have the proper values.
                // After: Deatach - modify - Attach, the original values contains
                // the values what had the object at the moment of the attach operation
                // that is why the values are read from the db.
                var dbValues = context.Entry(entry.Entity).GetDatabaseValues();
                CurrentValueRecord current = entry.CurrentValues;
                foreach (string propertyName in entry.GetModifiedProperties())
                {
                    string oldValue = null;
                    if (dbValues[propertyName] != null)
                        oldValue = dbValues[propertyName].ToString();

                    string newValue = null;
                    if (!current.IsDBNull(current.GetOrdinal(propertyName)))
                        newValue = current.GetValue(current.GetOrdinal(propertyName)).ToString();

                    if (oldValue != newValue)
                    {
                        this.AuditList.Add(CreateAuditLog(entry, propertyName, oldValue, newValue), entry);
                    }
                }
            }
            else if (entry.State == EntityState.Added )
            {
                this.AuditList.Add(CreateAuditLog(entry), entry);
            }
            else if(entry.State == EntityState.Deleted)
            {
                // after save the delete entry will have invalid id
                context.AuditLog.Add(CreateAuditLog(entry));
            }
        }
    }
}

public void AfterSaveChanges(DataContext context)
{
    if (this.AuditList.Count > 0)
    {
        UpdateKeys();
        context.AuditLog.AddRange(this.AuditList.Keys);
        context.SaveChanges();
    }
}
{% endhighlight %}

The full project can be found in my [EntityFameworkAudit][repo-url] repository.

[repoy-url]: https://github.com/szunyog/EntityFameworkAudit