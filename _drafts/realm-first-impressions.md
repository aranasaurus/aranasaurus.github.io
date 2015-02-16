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

TODO: Give brief high-level overview. Throw some code samples showing a data model class, creating, editing, and 
deleting an object of that class in/from a Realm.

# The App

The app is fairly simple. It's a ratings journal. You take a picture of a thing, rate it, comment on it, attach
some tags, maybe a location, and then save that to your device. Basically [Untappd](untappd) but for anything you
can take a picture of and without all the reliance on the social media aspects.

# Converting from CoreData

I recently got this app up and running as a side project to get some experience writing a full app in Swift. I
initially wrote the data layer in [CoreData][TODO] complete with using an [NSFetchedResultsController][TODO] for
my table view and everything. But last night I spent a couple of hours replacing all of that with [Realm Objects][TODO]
and [Realm Notification Blocks][TODO]. I had spent a few hours reading about Realm over the past week too and
poked at the docs some so it was a pretty straight forward process.

## Data Objects

I started off by replacing my `NSManagedObject` subclasses with `RLMObject` subclasses and getting rid of the stuff
I didn't need anymore: Bye-bye `ratings-app.xcdatamodeld`, `class func entityName() -> String` functions, and a bunch
of crap from my `AppDelegate`! Changing out the superclass for my data object classes wasn't difficult at all, mostly
just had to change from `@NSManaged` to `dynamic` for all of my properties and then use a [RLMArray][TODO] on the `Item`
class for `Tag`s and a [link back property][TODO] on `Tag` to get all `Item`s that link to it. The way they handle that is
pretty cool, and a bit easier to understand than the CoreData way, even if it is technically more code (and thankfully
less ~XML).

## Update ViewController Code

I tried to put most of my `CoreData` specific code into my `DataStore` class, but inevitably some of the `ViewController`
code ends up with some `CoreData` specific stuff in it, like a reference to the `NSManagedObjectContext`. Realm is no
different really. I keep a reference to the `RLMRealm` object on the `ViewController`s so that I can easily begin/commit
transactions when I need to edit the data (though I've now gone back and refactored that code to take it out of the VC,
more on that later). Realm is all transaction based, anytime you need to edit a property on an object that is stored in
a Realm, you have to do so within a Realm transaction. Thankfully these are very easy to start and commit:

{% highlight swift %}
let item = items[indexPath.row]
realm.beginTransaction()
item.comments = textField.text
realm.commitTransaction()
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

<!-- TODO: The above might be moved (at least partially) into the intro to Realm section at the beginning of the article. -->

The ease of this had me thinking about whether or not my separate `DataStore` class was even necessary. It really didn't
feel like it was... so I removed it. At first I just moved all of its implementation (that wasn't implemented as part of
Realm already) to just be inlined in the VCs, because these were just add/delete functions that with Realm just require you to
start a transaction, call a `RLMRealm` method, and then commit the transaction.

{% highlight swift %}
@IBAction func ratingChange(sender: UISlider) {
    if let realm = item.realm {
        realm.beginTransaction()
        item.rating = sender.value
        item.ratingDate = NSDate()
        realm.commitTransaction()
    }
}
{% endhighlight %}

<p style="line-height: 0.5em;" /> <!-- TODO: Get rid of this when I have some time to look at my theme a bit -->

After I had everything working I decided to refactor a little and try to make it so that ViewControllers could be oblivious
to the fact that this data was coming from Realm. First, I moved all data edits to methods on the data class that was being
edited. For example, the rating change method above was in my `DetailViewController` and my `ItemTableViewCell`, I changed
those to this:

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
