## Arities
[keyfn coll]
[keyfn comp coll]

## Documentation
{%raw%}
Returns a sorted sequence of the items in coll, where the sort
  order is determined by comparing (keyfn item).  If no comparator is
  supplied, uses compare. comparator must
  implement java.util.Comparator.
{%endraw%}
