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

******* INSERT sCREENSHOT OF DATASET

**Q.** Which is the difference between `less` and `less -S`?

If you want to see it more nicely and column aligned you can use `column -t` option:
```
less -S $GTEX | column -t | less -S
```

You can also check which are the first or the last rows of your document by using `head $yourfile` or `tail $yourfile`.

**Q.** Save the header line of the document in 1 column. Save it in a new document entitled: header.txt (hide this chunck)
```
head -n 1 $GTEX | tr "\t" "\n" > header.txt
```
**Q.** Apart from using the `head` command, which other alternatives could you use to obtain the header? (hide this chunck)

```
<details><summary>Code example</summary><p>
  
  grep "Name" $GTEX
  grep -v "ENS" $GTEX
  
</p></details>
```

```
grep "Name" $GTEX
grep -v "ENS" $GTEX
```

**Q.** How many genes are in this dataset
```
tail -n +1 $GTEX | wc -l
```




