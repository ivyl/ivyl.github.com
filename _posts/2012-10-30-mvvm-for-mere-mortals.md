---
layout: post
title: "MVVM For Mere Mortals"
tags: [c#, mvvm, dotnet, wpf]
---
{% include JB/setup %}

## Oh wow, C#? .NET? WPF?!

You won't hear from me on those topics very often. Recently I had to do few
thing in mentioned technology and I spent waaaaay too much time trying to
deal with it, and after all it rendered as simple.

## What's the fuss?

[Model View ViewModel](http://msdn.microsoft.com/en-us/magazine/dd419663.aspx)
- scary name. It's acronym is palindrome, which makes it even scarier if
you're palindromeophobe. All introductory video tutorials/blog posts that I
found on this topic are too complicated for their purpose. It's like big
conspiracy, where everyone using MS's technology want to feel adherence to
elite. Things that are easy and straightforward are described in such
manner that makes you sleepy (I. MUST. RESIST.) or way too hard to
understand in one reading. I'll try to fix it, especially for someone who
isn't from .NET aristocracy.

## MVVM explained

I won't cover full-blown MVVM framework. This post will be about the concept.
Treat it as introduction. We will write simple implementation from scratch to
get used to few new things and the new way of thinking.

Transition from Windows Form to WPF seems hard to most people. You can write
forms-style in WPF, but that's just wrong. We are given other, way more
powerful means. You will learn all about bindings, how ViewModel works, what
it does, and how to do things proper-ly. After that, I believe, you will have
intuitions good enough to dive into framework.

I assume that you know your MVC. It belongs to common knowledge nowadays and
it's well explored. If you don't know [grab your copy of Wikipedia's
article](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) on
it.

I treat MVVM as a close cousin of MVC. Model is Model, View is View, and
ViewModel, well, is something like Controller but not quite. The main
difference is that all interaction with ViewModel comes from View through
bindings. It means that ViewModel have some representation of data that is
bound to state of View's controls more or less directly. When something in View
changes it's reflected in ViewModel and we can react on that change.

I recommend to think of ViewModel as a mapping between Model representation of
data and what View expect. Since we don't keep in our Model nothing related to
the visualization, ViewModel deals with that, in this manner it's very similar
to controller. It also performs needed conversion back-and-forth to types that
given layer needs.

View are as stupid as possible. State of all controls is bound to
corresponding members of backing ViewModel and reflect changes in chosen
directions (from VM to V, in opposite direction or both ways).

ViewModel is active in some sort of way. We fire certain actions through it,
use timers, etc. After all it handles logic behind View.


Repeating one last time to sum up and memorize -  ViewModel keeps state, maps
our Model to View and perform actions on Model basing on interactions with
View (witch through data binding are instantly reflected in ViewModel) and
fires certain events.

Nothing more here. Yup, it's as simple as that.



## Guide me!

Having theory behind we should try it in practice. Let's start with
VisualStudio (latest Express should be fine) and fresh WPF project. I'll show
you how to bind data to some kind of fields like lists, how to perform actions
on state change and how to bind button to action.

### Preparation

I won't tell you how to create new window in your project or how to write
your model. Do something simple like User class that handles only few fields
and try things on it. Don't copy, analyze, use imagination and adapt.

### The ViewModel & The Binding

Let's start with some basic ViewModel. It should be just plain object with
public properties.

{% highlight c# %}
class MyViewModel {
    private User _user;
    public string UserName
    {
        get { return _user.Name; }
        set { _user.Name = value; }
    }
}
{% endhighlight %}

We can supply such class as
[Window's](http://msdn.microsoft.com/en-us/library/system.windows.window.aspx)
data context. You can do it from code when creating new window (i.e. in
app.xaml.cs);

{% highlight c# %}
Window userWindow = new UserWindow();
MyViewModel vm = new MyViewModel();
userWindow.DataContext = vm;
userWindow.Show();
{% endhighlight %}

Or, which is better when ViewModel doesn't need ultracustom, fancy initialization,
straight from window's XAML:

{% highlight xml %}
<Window x:Class="proj.View.UserWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="clr-namespace:proj.ViewModel"
        Title="Summary" Height="385" Width="257">
    
    <Window.DataContext>
        <vm:MyViewModel/>
    </Window.DataContext>
</Window>
{% endhighlight %}

Note registering new local XML namespace in root element, which points to our
ViewModel-containing C# namespace, which we use it to address our ViewModel.

This approach gives us advantage of easier binding, when editing XAML or
creating bindings through click-through GUI, IDE will aid us. I recommend
adding this temporally during view development even if you do things
differently.

Now we have our window prepared. Put some TextBox somewhere in the window,
select it and look at properties (pane in IDE). All of them have small square on the right of
their value. Click on it, and now you are given menu from which you can create
binding. When dealing with TextBox the sanest choice will be Text property.
Bind it to ViewModel's UserName and viola!

{% highlight xml %}
<TextBox skippedAttributesHere Text="{Binding UserName}"/>
{% endhighlight %}


### Button Command

Doing binding for buttons is bit more complicated. Property we should look at
is called Command. It expects implementation of
[ICommand](http://msdn.microsoft.com/en-us/library/system.windows.input.icommand.aspx)
interface. 
You can use
[one](http://msdn.microsoft.com/en-us/library/system.windows.input.routedcommand.aspx)
of the
[two](http://msdn.microsoft.com/en-us/library/system.windows.input.routeduicommand.aspx)
implementation we are supplied with. I recommend writing our own, simple one:

{% highlight c# %}
class Command : ICommand
{
    private Action exec;
    private Func<bool> canExec;

    public Command(Action exec, Func<bool> canExec)
    {
        this.exec = exec;
        this.canExec = canExec;
    }

    public bool CanExecute(object parameter)
    {
        if (canExec == null) return true;
        return canExec();
    }

    public event EventHandler CanExecuteChanged
    {
        add { if (canExec != null) CommandManager.RequerySuggested += value; }
        remove { if (canExec != null) CommandManager.RequerySuggested -= value; }
    }

    public void Execute(object parameter)
    {
        exec();
    }
}
{% endhighlight %}

It just wraps few delegates we can pass to it through constructor. **canExe**
returns true/false and tell us whether button should be active. **exec** is
proper action. If it's null then it's always active. Note the
**[CommandManager](http://msdn.microsoft.com/en-us/library/system.windows.input.commandmanager.aspx)**
usage. Its necessary to CanExecute work as expected (without it would be
called only upon creation). You may want to inspect it further.

Suppose in ViewModel we have following methods:

{% highlight c# %}
public void MyExecute() { /* ... */ }
public bool MyCanExecute() { /* ... */ }
{% endhighlight %}

We can then bind property given below to button's command:

{% highlight c# %}
public ICommand MyCommand { get { return new Command(MyExecute, MyCanExecute);  } }
{% endhighlight %}

If instead of CanExecute we supply constructor with *null*, then button is
active all the time.

### Collections

To reflect changes on collection immediately we use
[ObservableCollection](http://msdn.microsoft.com/en-us/library/ms668604.aspx).
It's very generic but fulfills it's purpose. You can find other
observable implementations in the vast plains of internet.

{% highlight c# %}
public ObservableCollection<MyUserWrapper> { get; set; }
{% endhighlight %}

Make sure that elements of list will have sane representation (*ToString*).

### When property changes...

To react to property change in simplest manner our class should inherit from
[DependencyObject](http://msdn.microsoft.com/en-us/library/system.windows.dependencyobject.aspx).
When it's done we can create
[DependencyProperty](http://msdn.microsoft.com/en-us/library/system.windows.dependencyproperty.aspx).
It should have name, know it's type and type of it's owner:

{% highlight c# %}
public static readonly DependencyProperty CurrentDateProperty 
     = DependencyProperty.Register("CurrentDate", typeof(DateTime), typeof(DateTimeViewModel),
     new PropertyMetadata(DateTime.Today, OnCurrentDateSelectionChanged));
{% endhighlight %}


We supplied it with metadata. Take a closer look at
[PropertyMetadata](http://msdn.microsoft.com/en-us/library/system.windows.propertymetadata.aspx),
it has some cool options. In our example we give it default value and give it
delegate to fire on change. We have to do it through static method:

{% highlight c# %}
public static void OnCurrentDateSelectionChanged(DependencyObject dobj, DependencyPropertyChangedEventArgs e)
{
    ((DateTimeViewModel)dobj).ViewModelInstanceMethod();
}
{% endhighlight %}

Since this whole staticness we get our ViewModel as parameter of type
DependencyObject. Some simple typecasting and we are done.

Now it's time to create property to bind. Using SetValue and GetValue it's
easy:

{% highlight c# %}
public DateTime CurrentDate {
    get { return (DateTime) GetValue(CurrentDateProperty); }
    set { SetValue(CurrentDateProperty, value); }
}
{% endhighlight %}

Writing all the code above might be bit painfull. *propdp* macro/snippet in
VisualStudio does most work for us.

### The End

That's all. I hope that you now have some intuition on this whole MVVM thing
and know how to bite it. Mind that those things are absolute basics and if
you interested in this topic you should use other resources.

Now you should see appeal of this approach. ViewModel holds all the logic and
is connected to Window through binding only. We can easily unit test the hell
out of it, which would be nearly impossible with forms-style implementation
(whether you write it in Windows Forms or WPF).

## Few Advices

They come from my little experience, and they come free.

### Expression Blend 4 is PAIN

Do not ever open existing project with it. It likes to mess with binding and
other stuff. It will waste a lot of your time. Luckily most complex project
won't even open. Do all bindings through VisualStudio and if you need something
that only Expression Blend does, do it on fresh project and then copy
corresponding XAML parts. It's most time efficient and it helps me keep my sanity.

### Put things where it belongs

What belongs to Model and what doesn't is more philosophical question. Everyone
have their opinion on it and most of them are valid in their contexts.

I found my own answer. If something does more or less complex action to model,
and to model only you should put it there. If it touches view in some way you
should split it. Put as much as you can in model (extraction of information,
etc.) and what deals with presentation - in ViewModel. If this task is
repetitive make it private method in there. After some time you should have
good intuition on it.


### Don't write it by hand

I've shown you how to write it from scratch. It has educational appeal but
there's easier way that somewhat reduces complexity. Use something like [MVVM
Light Toolkit](http://mvvmlight.codeplex.com/) next time. I heard that Prism
is also fine, but it's overall complexity scares me.  If you want write big
application in a manner described above - don't. After some time it would
become unmanageable. You don't write you MVC site from scratch either. WPF
gives us cool tools, but they only cover basics. Look for cool
macros/snippets too.

