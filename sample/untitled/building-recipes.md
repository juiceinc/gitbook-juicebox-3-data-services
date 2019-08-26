# Building Recipes

Warning

This help isn’t complete. It may even look terrible. If you want to work on it, see [How to Contribute](https://docs.juiceboxdata.com/projects/juicebox/topics/contributing.html#how-to-contribute). You can also ask for help in the Juice Slack \#documentation channel.

Recipes are the heart of data services for slices.

## dataservices.generator module[¶](building-recipes.md#dataservices-generator-module)

## Recipe Ingredients[¶](building-recipes.md#recipe-ingredients)

Ingredients are the raw material for making reusable SQLAlchemy queries. We define ingredients in the data service base class in the `dimension_shelf` and `metric_shelf`. These ingredients can then be used across all your data services.

```text
def MyBaseService(RecipeServiceBaseV3):
    dimension_shelf = {
        # A dictionary of reusable dimensions. The key is the "id"
        # of the dimension and the value is a Dimension object
    }

    metric_shelf = {
        # A dictionary of reusable metrics. The key is the "id"
        # of the dimension and the value is a Metric object
    }

    automatic_filter_keys = (...)
    # A tuple or list of values from the dimension_shelf
    # these will automatically be used for filters and show up in global filters
```

## Types of Ingredients[¶](building-recipes.md#types-of-ingredients)

DimensionDimensions are used for grouping data.LookupDimensionA dimension that takes an expression and a python dictionary to look the values up against. For instance, the following would change state abbreviations into full U.S. state names. If the state abbreviation wasn’t found “State not found” would be returned.

```text
LookupDimension(Census.state, {
  "AL": "Alabama",
  "AK": "Alaska",
  ...
}, default="State not found")
```

IdValueDimensionA dimension that takes two expressions. The first is used as the id of an item and the second as the displayed label of that thing. Imagine you had students. There might be multiple people named “Sally Baker”. These would have the same label but different ids and would show up as different values in the frontend.EncryptedDimensionA dimension that is encrypted at rest in the database. It will be decrypted using a secret when it is displayed in juicebox. Encrypted dimension injects a formatter in the first formatters position that decrypts the value. Any other formatters will have access to the decrypted value.MetricMetrics count and aggregate values.SumIfMetric\(conditional\_expression, summing\_expression\)Adds up the value in ```summing_expression ``if the ``conditional_expression``` is true. For instance, to count up the total amount of sales with status=’complete’, you could use:

```text
SumIfMetric(MyTable.status == 'complete', MyTable.sales
```

CountIfMetric\(conditional\_expression, counting\_expression\)Counts the number of values in `counting_expression` where `conditional_expression` is true.DivideMetric\(numerator\_expression, denominator\_expression\)Divides two expressions while avoiding problems that can occur when you divide by zero.

## Building a recipe[¶](building-recipes.md#building-a-recipe)

You will use your ingredients to build recipes to supply data for a slice.

```text
self.dimensions = ('age', 'city')
self.metrics = ('sales_dollars', 'sales_count')
self.recipe().metrics(*self.metrics).dimensions(*self.dimensions)\
   .filters(Filter(MyTable.state == 'Georgia'))
```

.metrics\(_list of metrics_\)Add one or more metrics to the recipe. Use the keys that you defined on the `metric_shelf`..dimensions\(_list of dimensions_\)Add one or more dimensions to the recipe. Use the keys from the `dimension_shelf`..filters\(_list of filters_\)Add one or more filters to the recipe. These can be keys from the `filter_shelf` or filter objects you create just for the recipe using Filter\(expression\).order\_by\(_list of dimensions or metrics to order by_\)A list of dimension or metric keys that determines the order results should appear in. The default is values appear in ascending order, but if you put a ‘-‘ sign before the key, it will sort in descending order..apply\_user\_filters\(_True or False_\)Should user filters be applied to this recipe. The default is True..apply\_stack\_filters\(_True or False_\)Should stack filters be applied to this recipe. The default is True..apply\_automatic\_filters\(_True or False_\)Should automatic filters be applied to this recipe. The default is True..include\_automatic\_filter\_keys\(_a list of keys_\)Limit the keys from _self.automatic\_filter\_keys_ that will apply for automatic\_filters to the provided list. There are two synonyms for this function _limit\_global\_filters\_to_ and _limit\_automatic\_filters\_to_.exclude\_automatic\_filter\_keys\(_a list of keys_\)Exclude certain keys in _automatic\_filter\_keys_ from being applied.compare\(_recipe, suffix_\)Add a comparison recipe. Comparison recipes will be matched to the base recipe by the `dimensions` in the comparison recipe. The `metrics` in the comparison recipe will be added to each row with a suffix..blend\(_recipe_\)Add a blend recipe. Blend recipes will be matched to the base recipe by looking at the tables used in the two recipes and matching constraints \(primary keys and foreign keys\). Only rows that are in both the blend recipe and the base recipe will be returned.full\_blend\(_recipe_\)Similar to `.blend()` but only rows that are in the base recipe will be returned. If there is no match in the blend recipe, the blend recipe values will be None.

## Using a recipe[¶](building-recipes.md#using-a-recipe)

Once you have a recipe you can either look at the values or use it to generate the slice response..all\(\)A list of all rows in the data. The row will have a property for each dimension and metric used in the recipe. The will also have a property `{{dimension_key}}_id` which is the id for for each dimension. If formatters are used there will be a `{{key}}_raw`, the unformatted value of that ingredient..one\(\), .first\(\)An object representing the first row returned..render\(\)Render the recipe in a proper response for this slice type..to\_sql\(\)See the SQL that this recipe generated.

## Debugging recipes[¶](building-recipes.md#debugging-recipes)

In juicebox3 apps, a frontend debugging view is available. This view is visible to superusers or if the recipe renders with parameter show\_debug=True. Client implementations like HealthStream can override FruitionUser.can\_see\_dataservice\_debug to customize who sees the debug views.

This debugging view is not available in HIPAA environments \(where `ALLOW_QUERY_CACHING=False` in settings\).

```text
def build_response(self):
    self.metrics = ('dollars', 'avg', 'gteed', 'per_gteed', 'age', 'yrs')
    self.dimensions = ('name', 'pos', 'team', 'start_year', 'end_year', 'free_agent', 'signed')
    recipe = self.recipe().metrics(*self.metrics).dimensions(*self.dimensions)
    self.response['responses'].append(recipe.render(show_debug=True))
```

The debug view is a modal dialog that shows automatic filters, custom filters, ingredients and the generated sql.

