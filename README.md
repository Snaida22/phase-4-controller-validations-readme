# Controller Validations

## Learning Goals

- Check the validity of a model in a controller
- Render a response with the error messages
- Use HTTP status codes to provide additional context

## Introduction

Now that we've seen how Active Record can be used to validate our data, let's
see how we can use that in our controllers to give our user access to the
validation errors, so they can fix their typos or other problems with their
request.

To get set up, run:

```sh
bundle install
rails db:migrate db:seed
rails s
```

## Manually Checking Validation

Up until this point, our `create` action has looked something like this:

```rb
# app/controllers/birds_controller.rb
def create
  bird = Bird.create(bird_params)
  render json: bird, status: :created
end
```

Let's add some validation to our `Bird` model, so that we don't end up with bad
data:

```rb
# app/models/bird.rb
class Bird < ApplicationRecord
  validates :name, presence: true, uniqueness: true
end
```

Now, if we try to create a bird using Postman with bad data, we've got a
problem!

```json
{
  "species": "Archilochus colubris"
}
```

Our server still returns a `Bird` object, but we can clearly see that it wasn't
saved successfully:

```json
{
  "id": null,
  "name": null,
  "species": "Archilochus colubris",
  "created_at": null,
  "updated_at": null,
  "likes": 0
}
```

From this process, we can tell:

- Our model validation prevented this bad data from being saved in the database
  (yay!)
- The response doesn't tell us anything about why the data wasn't saved (boo.)

To provide this additional context, we need to update our controller action to
change the response based on whether or not the bird was saved successfully.

```rb
def create
  bird = Bird.create(bird_params)
  if bird.valid?
    render json: bird, status: :created
  else
    render json: { errors: bird.errors }, status: :unprocessable_entity
  end
end
```

Now, we get a different response after sending that same request in Postman:

```json
{
  "errors": {
    "name": ["can't be blank"]
  }
}
```

From the controller, `bird.errors` will give a serializable object with all the
error messages from our Active Record validations.

We also included status code of [422 Unprocessable Entity][422], indicating this
was a bad request.

[422]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/422

We can take a similar approach to validation in our `update` method, since
validations will also run when a model is updated:

```rb
def update
  bird = Bird.find_by(id: params[:id])
  if bird
    bird.update(bird_params)
    if bird.valid?
      render json: bird
    else
      render json: { errors: bird.errors }, status: :unprocessable_entity
    end
  else
    render json: { error: "Bird not found" }, status: :not_found
  end
end
```

> At this point, our controller actions are getting a bit long! There are a few
> opportunities for DRY-ing up this code by creating some helper methods. See if
> you can identify which controller actions reuse the same logic, and extract
> that logic to helper methods.

## Summary

With model validations in place, we can help protect our database against bad
data. Active Record validations also help provide **error messages** to indicate
why a certain value wasn't considered valid data. We can access the model's
validity and error messages in the controller. By sending this data in the
response, we'll be able to provide additional context to our clients about what
went wrong with their request so they can fix it.