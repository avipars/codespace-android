## Overview

Android has now released a stable data-binding library which allows you to connect views with data in a much more powerful way than was possible previously. Applying data binding can improve your app by removing boilerplate for data-driven UI and allowing for two-way binding between views and data objects. 

The Data Binding Library is added as part of the Android Gradle plugin that is compatible with all recent Android versions.

### Setup

To get started with data binding, we need to make sure to upgrade to the latest version of the [[Android Gradle plugin|Getting-Started-with-Gradle#upgrading-gradle]].

To configure your app to use data binding, add the `dataBinding` element to your `app/build.gradle` file:

```gradle
apply plugin: 'com.android.application'

android {
    // Previously there
    // Add this next line
    dataBinding.enabled = true // <----
    // ...
```

### Eliminating View Lookups

The most basic thing we get with data binding is the elimination of `findViewById`. To enable this to work for a layout file, first we need to change the layout file by making the outer tag <layout> instead of whatever ViewGroup you use (note that the XML namespaces should also be moved):

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools">
    <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:paddingLeft="@dimen/activity_horizontal_margin"
            android:paddingRight="@dimen/activity_horizontal_margin"
            android:paddingTop="@dimen/activity_vertical_margin"
            android:paddingBottom="@dimen/activity_vertical_margin"
            tools:context=".MainActivity">

        <TextView
                android:id="@+id/tvLabel"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"/>

    </RelativeLayout>
</layout>
```

The `layout` container tag tells Android Studio that this layout should take the extra processing during compilation time to find all the interesting Views and note them for use with the data binding library.  

Essentially creating this outer `layout` tag causes a special reserved class file to be generated at compile time based on the name of the file.  For instance, `activity_main.xml` will generate a class called `ActivityMainBinding`.  Although this class is not generated at compile-time, it can still be referenced in Android Studio thanks to [integrated support](https://developer.android.com/topic/libraries/data-binding/index.html#studio_support) for data binding.
  
Now, we can use the `Binding` object to access the view.  In our activity, we can now inflate the layout content using the binding:
 
```java
public class MainActivity extends AppCompatActivity {
    // Store the binding
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Inflate the content view (replacing `setContentView`)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        // Store the field now if you'd like without any need for casting
        TextView tvLabel = binding.tvLabel;
        tvLabel.setAllCaps(true);
        // Or use the binding to update views directly on the binding
        binding.tvLabel.setText("Foo");
    }
}
```

Note that this binding object is generated automatically with the following rules in place:

 * Layout file `activity_main.xml` becomes `ActivityMainBinding` binding object
 * View IDs within a layout become variables inside the binding object (i.e `binding.tvLabel`)

The binding process makes a single pass on all views in the layout to assign the views to the fields. Since this only happens once, this can actually be faster than calling `findViewById`. 

Refer to [this guide for more details about bindings and inflating views](https://medium.com/google-developers/no-more-findviewbyid-457457644885#.p1ie9j52a).

### One Way Data Binding

The simplest binding is to automatically load data from an object into a view directly using a new syntax in the template. For example, suppose we want to display a user's name inside a `TextView`. Assuming a simple user class:

```java
public class User {
   public String firstName;
   public String lastName;
}
```

Next, we need to wrap our existing layout inside a `<layout>` tag to indicate we want data binding enabled:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        android:paddingBottom="@dimen/activity_vertical_margin"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/tvFullName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World" />

    </RelativeLayout>
</layout>
```

Next, we need to indicate that we want to load data from a particular object by declaring `variable` nodes in a `data` section of the `<layout>`:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
       <variable name="user" type="com.example.User"/>
    </data>
    <!-- ... rest of layout here -->
</layout>
```

The `user` variable within the data block describes a property that can now be used within this layout. Next, we can now reference this object inside of our views using `@{variable.field}` syntax such as:

```xml
<TextView
    android:id="@+id/tvFullName"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text='@{user.firstName + " " + user.lastName}' />
```

We can use conditional logic and other operations as part of the [binding expression language](https://developer.android.com/topic/libraries/data-binding/index.html#expression_language) for more complex cases. Finally, we need to assign that user data variable to the binding at runtime:

```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Inflate the `activity_main` layout
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        // Create or access the data to bind
        User user = new User("Sarah", "Gibbons");
        // Attach the user to the binding
        binding.setUser(user);
    }
}
```

That's all! When running the app now, you'll see the name is populated into the layout automatically:

<img src="http://i.imgur.com/Q8kscZQ.png" width="400" />

#### Controlling visibility

You can use binding expressions to control the visibility of an item.  

First, make sure you have access to the View properties (i.e. `View.INVISIBLE` and `View.GONE`) by importing
the object into the template.

```xml
<data>
   <import type="android.view.View" />
</data>
```

Next, you can test for a condition (i.e. first name is null) and determine whether to show or hide the text view  accordingly:

```xml
<TextView
   android:visibility="@{user.firstName != null ? View.VISIBLE: View.GONE}">

</TextView>
```

#### Image Loading

Unlike TextViews, you cannot bind values directly.  In this case, you need to create a custom  attribute.

First, make sure to define the `res-auto` namespace in your layout.  You cannot declare it in the `<layout>` outer tag so should put it on the outermost tag after the `<data>` tag:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
  
  <data>

  </data>

  <RelativeLayout xmlns:app="http://schemas.android.com/apk/res-auto">
  </RelativeLayout>
```

```xml
<ImageView app:imageUrl=“@{mymodel.imageUrl}”>
```

You then need to annotate a static method that maps the custom attribute:

```java

public class BindingAdapterUtils {
  @BindingAdapter({"bind:imageUrl"})
  public static void loadImage(ImageView view, String url) {
     Picasso.with(view.getContext()).load(url).into(view); 
  }
}
```

### Two Way Data Binding

If you want to have a two-way binding between the view and the data source, check out this [handy 2-way data binding tutorial](https://medium.com/@fabioCollini/android-data-binding-f9f9d3afc761#.6h923gix6).

### Troubleshooting

* If you see an error message such as `**.**.databinding does not exist`, it's likely that there is an error in your data binding template.  Make sure to look for errors (i.e. forgetting to import a Java class when referencing it within your template).

* If you are using the data binding library with the `android-apt` plugin, you may need to add a reference to the library according to this [issue](https://bitbucket.org/hvisser/android-apt/issues/38/android-apt-breaks-brand-new-data-binding#comment-18504545).

```gradle
    apt 'com.android.databinding:compiler:1.0-rc0'
```

## References

* <https://developer.android.com/topic/libraries/data-binding/index.html>
* <https://medium.com/google-developers/no-more-findviewbyid-457457644885#.p1ie9j52a>
* <https://realm.io/news/data-binding-android-boyar-mount/>
* <https://www.captechconsulting.com/blogs/android-data-binding-tutorial>
* <http://www.survivingwithandroid.com/2015/08/android-data-binding-tutorial-2.html>