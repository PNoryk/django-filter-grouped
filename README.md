# django filter groups

This package groups filters generated by django-filter.

The main reason to use it - show only selected filters.

It groups filters by their `field_name`. 
The filter lookup will be subtracted from `filter_name`. 
That's why `int_field` and `int_field__isnull` in example above, will be placed in ine group

```python
class MyFilterSet(django_filters.FilterSet):
    int_field = django_filters.NumberFilter()
    int_field__isnull = django_filters.NumberFilter(lookup_expr="isnull")
```

A group name resolution goes the next way:
- finding `filter_group_label` in any filter in current group
- if multiple were found, the first would be selected
  - it's not good idea to add multiple `filter_group_label`'s
  - the ways to add `filter_group_label` you can find [here](#change_group_name)
- if `filter_group_label` wasn't added, it's finding label declared in a filter with `exact` lookup expression
- if it isn't presented, it'll get the first declared filter for this group and take its label
- if no label is found, it'll get the field verbose_name from model
- otherwise, it'll set `"[invalid name]"` group name, and it won't be able to choose such group 

## how to use
```
pip install django-filter-groups
```
add to your `INSTALLED_APPS` after `django-filter`
```python
INSTALLED_APPS = [
  ...
  "django_filters",
  "django_filters_groups",
  ...
]
```

use it in your template:
- add `{% add_select_filter_form_to_context %}` to the top of your template. 
  It allows you to place `{{ select_filter_form.media }}` anywhere you want
- add `{% filters_by_groups %}`
- add `{{ select_filter_form.media }}`

If FilterSet name is not 'filter' -> add filterset to both tags \
`{% add_select_filter_form_to_context my_custom_filterset %}`\
`{% filters_by_groups my_custom_filterset %}`
or filterset name
`{% filters_by_groups "my_custom_filterset" %}`
`{% add_select_filter_form_to_context "my_custom_filterset" %}`

## warning
It's necessary to add values to `exact` and `iexact` [VERBOSE_LOOKUPS](https://django-filter.readthedocs.io/en/stable/ref/settings.html?highlight=verbose_lookups#filters-verbose-lookups).

Filter choice will be empty if the lookups isn't presented

## default settings

```
# django settings
FILTERS_GROUPS_SELECT_FILTER_FORM_LABEL = "Select a label"
```
```js
// static/django_filters_groups/filter-defaults.js
let filterDefaults = {
  filterWrapperSelector: "p", // p is necessary when you use {{ form.as_p }}
  submitOnFilterDelete: false,
};
```

## <a name="change_group_name"></a> change group name
You can set group name directly in 2 ways:
- add `filter_group_label` to your filter
```python
class FFieldCountFilter(django_filters.NumberFilter):
    filter_group_label = "custom_group_label"
```
- use `get_filter_class_with_custom_label`
```python
from django_filters_groups.utils import get_filter_class_with_group_label

class MyFilterSet(django_filters.FilterSet):
  custom_filter = get_filter_class_with_group_label(django_filters.NumberFilter, "custom_group_label")(method="custom_filter_method")
```
