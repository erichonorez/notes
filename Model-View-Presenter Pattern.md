# Model-View-Presenter Pattern

## Introduction
The goal of this pattern is to separate the concerns of the presentation layer into three roles:

* the **model** moves the data to display from the presenter to the view
* the **view** binds the model properties to the visual components and delegates user interactions to the presenter
* the **presenter** handles user interactions and drives the view in order to react to events

Separating concerns improve the **testability** of the system. The system would be fully testable if all specifications could be asserted on the presenter and on the model.

![MVP overview](http://i.imgur.com/uxBtOaT.png)

In order to deeper explain the pattern let's dive into some examples. These ones comes from a fictive blog application and are deliberately simple.

## Display data

The first example is very simple. The ``BlogView`` view just renders the blog having the identifier specified in the URL of the page. E.g. ``http://my.blog.com/blogs/0f4d110b-3137-4bfb-8e2d-64099c9c3401``.

So in order to achieve it the following sequence of operations is executed:

1. the application routes the HTTP request to the ``BlogView`` view;
3. the view tells the presenter to initialise it;
4. the presenter fetch the blog with the identifier from the business layer;
5. it then creates a ``BlogViewModel`` model with the blog entity fetched from the business layer. This presentation model holds all information rendered by the view or needed to render it;
6. finally the presenter tells the view to bind its components to the model;

![Request sequence](http://i.imgur.com/mcglOW3.png)

Here is the unit test defining the specification:
```java
    @Test
    public void itShouldDisplayBlogById() {
        // Given a blog with id "0f4d110b-3137-4bfb-8e2d-64099c9c3401" exists
        String aBlogId = "0f4d110b-3137-4bfb-8e2d-64099c9c3401";

		    // ... BlogRepository mocking and stubbing...

        // When the user goes to the blog page
        BlogPresenter presenter = new BlogPresenterImpl(blogRepository);
        BlogView view = mock(BlogView.class);
        presenter.initView(view, aBlogId);

        // Then the blog is displayed
        ArgumentCaptor<BlogViewModel> argument
	        = ArgumentCaptor.forClass(BlogViewModel.class);

        verify(view).bind(argument.capture());

        BlogViewModel viewModel = argument.getValue();

        assertThat(viewModel.title())
            .isEqualTo(aTitle);

        assertThat(viewModel.content())
            .isEqualTo(aContent);
    }
```

The APIs of the presenter and the view

```java
/**
 * Controls the display of a blog.
 */
public interface BlogPresenter {

    /**
     * Initialise the view for the specified blog identifier
     *
     * @param view the calling view
     * @param aBlogId the identifier of the blog to diplay
     */
    void initView(BlogView view, String aBlogId);

}
```
When the view is ready to be rendered it calls the ``initView`` method of its presenter passing its reference and the query parameters. In that case the only query parameter is the blog identifier.

```java
/**
 * Displays a blog to the user
 */
public interface BlogView {

    /**
     * Bind properties of the BlogViewModel to visual components.
     */
    void bind(BlogViewModel viewModel);

}
```

After having fetch data from a persistence storage the presenter builds a ``BlogViewModel``  model and tells the view to bind visual components by calling the ``bind`` method.

Here is the implementation of the presenter:

```java
public class BlogPresenterImpl implements BlogPresenter {

    private BlogRepository blogRepository;

    public BlogPresenterImpl(BlogRepository blogRepository) {
        this.blogRepository = blogRepository;
    }

    public void initView(BlogView view, String aBlogId) {
        JsonObject blog = this.blogRepository.findById(aBlogId);

        view.bind(
            new BlogViewModel(
                blog.get("title").getAsString(),
                blog.get("content").getAsString()));
    }

}
```

###  Handle request parameters

``BlogPresenter#initView`` receive all necessary arguments to create a ``BlogViewModel``. It must receive all information it needs in order to do its job.

If the information the view displays depends is internationalized a locale must be passed to the presenter. If the display depend on the current user authorization the  presenter must receive the current user in parameter.

Other common parameters may be:
 * The query parameters
 * The pagination parameters (offset, limit)

Imagine that the blogs are available on several language and depends on the current user, the signature of the ``initView``method might be:

```java
public class BlogListPresenter {
  /**
   * @param aView the calling view
   * @param aLocale the locale in which the blogs must be displayed
   * @param aUser the logged user
   * @param offset the number of first blogs to skip
   * @param limit the number of blogs to return
   */
	public initView(
    BlogListView aView,
    Locale aLocale,
    User aUser,
    int offset,
    int limit) {

		Model model = //fetch data from db and do some computations
		// the presenter tell the view to bind the model
		view.bind(model);
	}
}
```

In that case the model would contain the list of blogs with pagination information:
* the total number of pages
* the current page

The **model** contains all necessary information to render the view without having logic in the view. The view only knows how to render things and not what to render.

## Tell the view to do
The presenter must tell the view what to do. The view should never take any decision to do something when an event occurs.

### Handling exceptions
It is for that reason that exceptions should never be handled by the view. The presenter should catch exceptions and tell the view what to do.

Let's imagine the case where the visitor tries to access a blog that does not exists. In that scenario the application must:

1. notify the user the blog he tried to access does not exist;
2. redirect the user to the list of blogs.

Here are the unit tests asserting the specifications:

```java
public static class WhenBlogDoesNotExist {

        @Test
        public void itShouldNotifyTheUser()
	        throws BusinessObjectNotFoundException {
            // Given a blog with id "0f4d110b-3137-4bfb-8e2d-64099c9c3401" does not exist
            String aBlogId = "0f4d110b-3137-4bfb-8e2d-64099c9c3401";

            BlogRepository blogRepository = mock(BlogRepository.class);
            when(blogRepository.findById(aBlogId))
	            .thenThrow(new BusinessObjectNotFoundException());

            // When the user goes to the blog page
            BlogPresenter presenter = new BlogPresenterImpl(blogRepository);
            BlogView view = mock(BlogView.class);
            presenter.initView(view, aBlogId);

            verify(view).notifyError(anyString());
        }

        @Test
        public void itShouldRedirectTheUserToBlogList() throws
	        BusinessObjectNotFoundException {
            // Given a blog with id "0f4d110b-3137-4bfb-8e2d-64099c9c3401" does not exist
            String aBlogId = "0f4d110b-3137-4bfb-8e2d-64099c9c3401";

            BlogRepository blogRepository = mock(BlogRepository.class);
            when(blogRepository.findById(aBlogId))
	            .thenThrow(new BusinessObjectNotFoundException());

            // When the user goes to the blog page
            BlogPresenter presenter = new BlogPresenterImpl(blogRepository);
            BlogView view = mock(BlogView.class);
            presenter.initView(view, aBlogId);

            verify(view).navigateToBlogList();
        }

    }
```

If we are looking at the business component that fetches the blog by its id:

```java
/**
 * Re-creates Blog from storage.
 */
public interface BlogRepository {

    /**
     * Fetch a blog form persistence storage or throw
     * BusinessObjectNotFoundException
     * if not found.
     */
    public JsonObject findById(String aBlogId)
	    throws BusinessObjectNotFoundException;

}
```

So the goal is to avoid the exception to leak into the view and give the view to responsibility to handle it. So the ``initView`` method of the presenter:


```java
public class BlogPresenterImpl implements BlogPresenter {

	// ... code

	public void initView(BlogView view, String aBlogId) {
       JsonObject blog;
       try {
           blog = this.blogRepository.findById(aBlogId);

           view.bind(
               new BlogViewModel(
                   blog.get("title").getAsString(),
                   blog.get("content").getAsString()));

       } catch (BusinessObjectNotFoundException e) {
           view.notifyError("The blog does not exist");
           view.navigateToBlogList();
       }
   }

   // ... code
}

```

## Internationalization

All data coming from the presenter that need to be translated when displayed must be returned to the view translated.

Data that does not come from the presenter are internationalized by the view. E.g. labels and all other static values.


## Handling forms

Once again the goal is to be able to test form validation by unit testing. So the validation must be on the presenter side.

Once the user what's to create a blog or to update existing one here are the rules applied:

- The title of the blog
	- cannot be null
	- length must be between 1 and 100
- The content of the
	- cannot be null
	- length must be between 1 and 2048

We are going to use the **Java Bean Validation API** in order to validate the form object.

```java

public class BlogForm {

    @NotNull
    @Size(min = 1, max = 100)
    public String title;

    @NotNull
    @Size(min = 1, max = 2048)
    public String content;

}
```

Here are the unit tests asserting the specifications:

```java
public static class WhenFormIsNotValid {

        @Test
        public void itShouldDisplayErrorWhenTitleIsEmpty() {
            // When the title of the form has not been filled in
            BlogForm form = new BlogForm();

            // Then the error must be displayed to the user
            Validator validator = Validation
                .buildDefaultValidatorFactory().getValidator();

            BlogEditionPresenter presenter
	            = new BlogEditionPresenterImpl(validator, mock(BlogRepository.class));
            BlogEditionView view = mock(BlogEditionView.class);

            presenter.save(view, form);

            Class<Set<ConstraintViolation<BlogForm>>> argumentClass =
                (Class<Set<ConstraintViolation<BlogForm>>>) (Class) Set.class;

            ArgumentCaptor<Set<ConstraintViolation<BlogForm>>> argument =
                ArgumentCaptor.forClass(argumentClass);

			// Then the application should show errors to user
            verify(view).displayErrors(argument.capture());

            Set<ConstraintViolation<BlogForm>> validations = argument.getValue();
            BWValidationAssert.fails(validations, "title");
        }

    }


    public static class WhenFormIsValid {

        @Test
        public void itShouldNotifyUser() {
            // When the blog form is valid
            BlogForm form = new BlogForm();
            Validator validator = mock(Validator.class);
            when(validator.validate(form))
                .thenReturn(new HashSet<ConstraintViolation<BlogForm>>());

            BlogRepository blogRepository = mock(BlogRepository.class);
            when(blogRepository.save(any(JsonObject.class)))
                .thenReturn(UUID.randomUUID().toString());

            BlogEditionPresenter presenter
	            = new BlogEditionPresenterImpl(validator, blogRepository);
            BlogEditionView view = mock(BlogEditionView.class);

            presenter.save(view, form);

            // Then the application should notify the user
            verify(view).notifySuccess(anyString());
        }

        @Test
        public void itShouldRedirectTheUserToTheShowBlogView() {
            // When the form blog is valid
            BlogForm form = new BlogForm();
            Validator validator = mock(Validator.class);
            when(validator.validate(form))
                .thenReturn(new HashSet<ConstraintViolation<BlogForm>>());

            BlogRepository blogRepository = mock(BlogRepository.class);
            String aBlogId = UUID.randomUUID().toString();
            when(blogRepository.save(any(JsonObject.class)))
                .thenReturn(aBlogId);

            BlogEditionPresenter presenter = new BlogEditionPresenterImpl(validator, blogRepository);
            BlogEditionView view = mock(BlogEditionView.class);

            presenter.save(view, form);

            // Then the application should redirect the user
            verify(view).navigateToShowBlogView(aBlogId);
        }
    }
```

In order to execute the scenario the ``view`` must have the following API:

```java
public class BlogEditionView {

  /**
     * Display the errors to the user
     */
    public void displayErrors(Set<ConstraintViolation<BlogForm>> capture);

    /**
     * Show a success notification message
     */
    public void notifySuccess(String message);

    /**
     * Redirect the user to display a blog with the given identifier
     */
    public void navigateToShowBlogView(String aBlogId);
}

```

And here is the presenter implementation:

```java
public class BlogEditionPresenterImpl {

    // ...

	public void save(BlogEditionView view, BlogForm form) {
        Set<ConstraintViolation<BlogForm>> validations = this.validator.validate(form);
        if (!validations.isEmpty()) {
            view.displayErrors(validations);
            return;
        }

        JsonObject json = new JsonObject();
        json.addProperty("title", UUID.randomUUID().toString());
        json.addProperty("content", UUID.randomUUID().toString());
        String blogId = this.blogRepository.save(json);

        view.notifySuccess("Blog edited!");
        view.navigateToShowBlogView(blogId);
    }

	// ...
}

```

## The view

As previously said the goal is to minimize the responsibilities of the view in order to increase the test coverage of the application. Moreover the thinner the view layer is the easier it is possible the make the view evolve (e.g. framework migration). The responsibilities of the view must be limited to data formatting and binding.

There is no technical constraints on the method names but in order to harmonies things here are some guidelines.

``bind`` method should be used in order to let the presenter to tell the view to bind the model. So the visibility of this method must be ``public``.

``buildSkeleton`` method should be used in order to create all components that does not depends on the model to be displayed. For instance all static texts and labels must be created in that method. These static values must be also internationalised in that method.

``updateLabels`` is used in order to refresh all static values when the locale of the application changes.

##The presenter
As for the view there is not technical constraints on the method names, only promoted practices.

``initView`` method should be used in order to let the view notify the presenter that it is going to be displayed. If the presenter must provide a model to the view it will pass it by calling ``bind``.

### Boundaries
- Only primitive types  or data objects cross the boundaries from the view to presenter
- Only primitive types or model cross the boundaries from the presenter to the view

##The model
**The model is not the domain model!** It is a presentation model existing only to holds all necessary information to render the view.

## Notes

All these example have been developed following the TDD methodology.

## Next steps
* View gives access to a navigator object
* View gives access to a notifier object
* View gives access to an eventBus in order to allow inter-view communication


----------


## Resources
http://martinfowler.com/eaaDev/SupervisingPresenter.html
