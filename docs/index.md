# Part 1. Bash language



To start download the [working document](https://github.com/pesteller/Intro2Bioinfo_course/blob/main/GTEx_v8_gene_median_tpm.gct).
This is expression data from GTEx version 8.


## Exploring the dataset

Save the dataset in a bash variable:
```
GTEX=GTEx_v8_gene_median_tpm.gct
```

How does our dataset look like?
```
less -S $GTEX
```

**Q1.** Which is the difference between `less` and `less -S`?

If you want to see it more nicely and column aligned you can use `column -t` option:
```
less -S $GTEX | column -t | less -S
```

You can also check which are the first or the last rows of your document by using `head $yourfile` or `tail $yourfile`.
**Q2.** How many lines does `head` show by default? Use `wc -l` to check it.

```
{
  head $GTEX | wc -l
}
```
