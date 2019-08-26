# Anonymizing Ingredients

Juicebox 3 supports anonymizing ingredients consistently throughout a stack.

## What do we mean by anonymization?[¶](anonymizing-ingredients.md#what-do-we-mean-by-anonymization)

Anonymization changes the displayed labels of dimensions to something generic . We do this to allow you to demonstrate an application \(using real data!\) but the actual names of the products or people you’re telling a story about are changed to something unguessable.

Anoymization uses real Metric data and the real ids of the Dimensions you’re using, so numbers are realistic and the app works normally. If the `is_demo_user` flag on a Juicebox user is set to `True` they’ll see all data anonymized. This is how we allow someone to demo or test a Juicebox app without being able to see the full data behind it.

## How do I do it?[¶](anonymizing-ingredients.md#how-do-i-do-it)

Anonymization is accomplished by adding a `anonymizer=<function name>` argument on the Dimension in the `dimension_shelf`.

You can mark recipes that should produce anonymized output by adding the `.anonymize(True)` method to any recipe. When your app is visited by a user who has the `is_demo_user` flag turned on, all recipes will automatically get `.anonymize(True)`.

## How it Works[¶](anonymizing-ingredients.md#how-it-works)

If your recipe specifies that it should be anonymized when it is processing the values for each ingredient, it will check to see if the ingredient is set to be anonymized and if it has an anonymizer. If both of these are True, it will check cache for a key with the stack and ingredient names. If that key is not found, it will run the anonymizer on the ingredient and store it in the cache. Then it will return the cache value. This is done because it allows the ingredient to be used in multiple anonymized recipes within the stack and get the same anonymized value consistently. Anonymized values are stored in the cache for 2 hours.

## Example[¶](anonymizing-ingredients.md#example)

In my stack’s dataservice python file, I have added a function to anonymize the states by reversing their name with the following function:

```text
def state_flipper(value):
    return value[::-1]
```

Then changed the state ingredient in our dimension\_shelf from the previous example to:

```text
'state': Dimension(Census.state, singular='State', plural='States',
                   format="", anonymizer=state_flipper),
```

Now the ingredient will be anonymized in any recipe that is set to be anonymized. We can test it by creating a service to power a Card slice, and set it to be anonymized **explicitly**.

```text
class CardV3Service(CensusService):
    def build_response(self):
        self.metrics = ('popdiff',)
        self.dimensions = ('state',)
        recipe = self.recipe().metrics(*self.metrics).dimensions(
            *self.dimensions).anonymize(True)

        self.response['responses'].append(recipe.render('States'))
```

This will result in every state value being reversed in the output.

Note

Most of the time you don’t want to anonymize a recipe yourself with `recipe.anonymize(True)`. You’ll rely on the `is_demo_user` flag to automatically turn on `anonymize(True)` for all recipes.

