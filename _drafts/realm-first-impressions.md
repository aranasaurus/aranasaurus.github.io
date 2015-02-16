---
layout: post
title: "Realm First Impressions"
request_feedback: false
---

I've been interested in [Realm][realm] for quite a while now, but hadn't really had a chance to look at it real
closely. But it has popped up in different ways a bunch of times over the past couple of weeks, and I have a [personal
project][ratings-app] that I think would be a good fit to try it out in. So that's what I did this weekend, and I
thought it'd be good to jot down my experience with it and my outlook on its future in my app.

<!-- more -->

# Wait, Realm? What's that?

According to their [website][realm]: 

> Realm is a mobile database: a replacement for SQLite & Core Data Realm can save you thousands of lines of code & weeks of work, and lets you craft amazing new user experiences.

What attracts me to Realm is that it lets you build your data model using your objects and arrays in code. There's
no separate object model schema file that you have to keep up to date separately from your classes that you use
to actually interact with the data. Your classes are your schema. There's also very little boiler plate, which, in
comparison to Core Data is straight up amaze-balls. It's also thread-safe. And queryable. And fast. And just dead
simple to set up and use.

Check this out:

{% highlight swift %}
class Dog: RLMObject {
    dynamic var name = ""
    dynamic var age = 0
}

let fido = Dog()
fido.name = "Fido"
fido.age = 5

let rex = Dog()
rex.name = "Rex"
rex.age = 1

let realm = RLMRealm.defaultRealm()
realm.beginWriteTransaction()
realm.addObjects([fido, rex])
realm.commitWriteTransaction()

// Rex and Fido are now persisted to disk and changes to their properties must be done within a
// Realm transaction, and are persisted immediately after the transaction is commited including
// from other threads!

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
    RLMRealm.defaultRealm().transactionWithBlock() {
        for r in Dog.objectsWhere("age < 2") {
            let d = r as Dog
            d.name = "\(d.name), The Puppy"
        }
    }
    
    let rex = Dog.objectsWhere("name contains 'Rex'").firstObject() as Dog
    println(rex.name) // "Rex, The Puppy"
}

{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

# The App

The app is fairly simple. It's a ratings journal. You take a picture of a thing, rate it, comment on it, attach
some tags, maybe a location, and then save that to your device. Basically [Untappd](untappd) but for anything you
can take a picture of and without all the reliance on the social media aspects.

The data model for the app is super simple. There are two objects:

  1. Item - This is the main class it has properties for title, rating, date, comments, image, and tags.
  2. Tag - This class just has a property for the tag name and an inverse relationship which backlinks to all `Items` that use the tag.

# Converting from CoreData

I recently got this app up and running as a side project to get some experience writing a full app in Swift. I
initially wrote the data layer in [CoreData][TODO] complete with using an [NSFetchedResultsController][TODO] for
my table view and everything. But last night I spent an hour or two replacing all of that with [Realm Objects][TODO]
and [Realm Notification Blocks][TODO]. The docs are pretty good and the tech itself is very straight forward,
so there wasn't a lot of bashing my head into the wall... which might sound odd, if you've spent any time with
CoreData.

Adding the Realm framework to the project is super simple:

  1. Drag n drop the framework file into the project and link it to the app target.
  2. Link against libc++.dylib
  3. Copy over the RLMSupport.swift file for a couple of Swift niceties.

See their [docs][TODO] for details and other configuration info.

## Data Objects

I started off by replacing my `NSManagedObject` subclasses with `RLMObject` subclasses and getting rid of the stuff
I didn't need anymore: Bye-bye `ratings-app.xcdatamodeld`, `class func entityName() -> String` functions, and a bunch
of crap from my `AppDelegate`! Changing out the superclass for my data object classes wasn't difficult at all, mostly
just had to change from `@NSManaged` to `dynamic` for all of my properties and then use a [RLMArray][TODO] on the `Item`
class for `Tags` and a [backlink][TODO] on `Tag` to get all `Items` that link to it. The way they handle that is
pretty cool, and a bit easier to understand than the CoreData way, even if it is technically more code (and thankfully
less `notcode`).

{% highlight swift %}
class Item: RLMObject {
    dynamic var guid = NSUUID().UUIDString
    dynamic var imagePath = "default.png"
    dynamic var rating: Float = 0
    dynamic var ratingDate = NSDate()
    dynamic var comments = ""
    dynamic var name = ""
    dynamic var tags = RLMArray(objectClassName: Tag.className())

    override class func primaryKey() -> String {
        return "guid"
    }

    var image: UIImage? {
        get {
            if let img = UIImage(contentsOfFile: imagePath) {
                return img
            }
            return UIImage(named: "default.png")
        }
    }

    // Other stuff...
}

class Tag: RLMObject {
    dynamic var name = ""
    var items: [Item] {
        return linkingObjectsOfClass(Item.className(), forProperty: "tags") as [Item]
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

## Update ViewController Code

I tried to put most of my `CoreData` specific code into my `DataStore` class, but inevitably some of the `ViewController`
code ends up with some `CoreData` specific stuff in it, like a reference to the `NSManagedObjectContext` and calls to its
`save` method. Realm is no different really. I keep a reference to the `RLMRealm` object on the `ViewControllers` so that
I can easily fire off transactions when I need to edit the data (though I've now gone back and refactored that code to
take it out of the VC, more on that later). Realm is all transaction based, anytime you need to edit a property on an
object that is stored in a Realm, you have to do so within a Realm transaction. Thankfully these are very easy to start
and commit:

{% highlight swift %}
// Other ViewControllery stuff..

let item = items[indexPath.row]
realm.transactionWithBlock {
    item.comments = textField.text
}

// Other ViewControllery stuff..
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

The ease of this had me thinking about whether or not my separate `DataStore` class was even necessary. It really didn't
feel like it was... so I removed it. At first I just moved all of its implementation (that wasn't implemented as part of
Realm already) to just be inlined in the VCs, because these were just add/delete functions that with Realm just require you to
start a transaction, call a `RLMRealm` method, and then commit the transaction.

{% highlight swift %}
@IBAction func ratingChange(sender: UISlider) {
    if let realm = item.realm {
        realm.transactionWithBlock() {
            item.rating = sender.value
            item.ratingDate = NSDate()
        }
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

After I had everything working I decided to refactor a little and try to make it so that ViewControllers could be oblivious
to the fact that this data was coming from Realm. First, I moved all data edits to methods on the data class that was being
edited. For example, the rating change method above was in my `DetailViewController` and my `ItemTableViewCell`, I changed
those to the following:

{% highlight swift %}
// In the VCs:
@IBAction func ratingChange(sender: UISlider) {
    item.updateRating(sender.value)
}

// In Item.swift:
func updateRating(rating: Float) {
    if let r = realm {
        realm.beginWriteTransaction()
    }

    self.rating = rating
    ratingDate = NSDate()

    if let r = realm {
        realm.commitWriteTransaction()
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

This moved the data persistence logic out of my VC and into my data layer. I like that! So I did it some more. I added
instance methods to `Item` so that instances could `save()` (which is an [upsert][TODO]) and `delete()` themselves from
whatever `Realm` they are attached to. This works great for my situation as I'm not using multiple Realms, if I were these
would have to be made class methods and would have to take the `Realm` to save/delete them to/from as a parameter, which
would kind of defeat the purpose.

## Replace NSFetchedResultsController Functionality

Replacing the [NSFetchedResultsController][TODO] functionality mostly consisted of *removing* code and adding one bit
to the `viewDidLoad` method:

{% highlight swift %}
token = realm.addNotificationBlock { (notification: String!, rlm: RLMRealm!) in
    if notification == RLMRealmDidChangeNotification {
        self.tableView.reloadData()
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

This is *much* simpler and more straight forward than the 80 or so lines of code it replaced although it does lose out on
some functionality: there's no animations for changes that are made to the tableView's rows. This functionality is a
large part of what the removed code handled and I think it'll find it's way back in once Realm supports more [fine-grained
notifications][TODO], which is apparently high on their list of things to get implemented.

# Migrations

I experimented a little bit with Migrations as I was fiddling with the types of the properties I was storing and I like
what they've done here. It's really easy to understand and seemed powerful enough for anything I could think up. Here's
the tiny migration I made to change a property from being stored as `Float` to `Double`:

{% highlight swift %}
// In AppDelegate's didFinishLaunching...
RLMRealm.setSchemaVersion(1, forRealmAtPath: RLMRealm.defaultRealmPath()) { migration, oldSchemaVersion in
    if oldSchemaVersion < 1 {
        migration.enumerateObjects(Item.className()) { oldObject, newObject in
            newObject["rating"] = Double(oldObject["rating"] as Float)
        }
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

# Some extra Realm Extensions

I made myself a few little helpers and put them in a `RealmExtensions.swift` file. There's not very many so I thought
it'd be nice to go over them in detail here.

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

  * RLMResults - `arraySortedByProperty(:ascending:)`
    I saw this method name in the docs somewhere, but couldn't find it in the header file, so I made an extension with a
    basic implementation of what I thought it probably did. Moreover, what I *wanted* a method with that name to do. It
    looks a little silly, but it's very handy in practice, as it takes a `RLMResults` object which is Array-like, but not
    actually an Array, and returns a `[RLMObject]` which is, in fact, an Array.

{% highlight swift %}
```
extension RLMResults {
    func arraySortedByProperty(propertyName: String!, ascending: Bool) -> [RLMObject] {
        return map( self.sortedResultsUsingProperty(propertyName, ascending: ascending) ) { (r: RLMObject) -> RLMObject in
            return r
        }
    }
}
```
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

    * RLMObject - save() / delete()
      These are instance methods that make it so that an `RLMObject` can save/delete itself in the `RLMRealm` in its `realm`
      property or the default Realm if it doesn't have one. Note that `add()` is doing an `addOrUpdateObject()` so it
      requires the RLMObject to have a [primaryKey][TODO] set. Also note that `delete()` is a no-op if the object is
      not currently in a Realm, because what else is it supposed to do?

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

{% highlight swift %}
extension RLMObject {
    func save() {
        if let r = realm ?? RLMRealm.defaultRealm() {
            r.beginWriteTransaction()
            r.addOrUpdateObject(self)
            r.commitWriteTransaction()
        }
    }

    func delete() {
        if let r = realm {
            r.beginWriteTransaction()
            r.deleteObject(self)
            r.commitWriteTransaction()
        }
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

    * RLMObject - addNotificationBlock() / removeNotification()
      These were just wrappers to make it so that I could add/remove Notification blocks to the default realm via my data
      object classes. They are simply one-line wrappers that forward on to their corresponding methods on the default
      `RLMRealm`.

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

# Final Thoughts

TODO:

[realm]: http://realm.io "Realm"
[ratings-app]: https://github.com/aranasaurus/ratings-app "Ratings App"
[untappd]: todo: "Untappd"
[TODO]: http://istillneedtomakethisarealurl.com/ "TODO: Fill in this link!"
