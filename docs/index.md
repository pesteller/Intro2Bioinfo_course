# Part I. Basics of bash
*Materials created by Marina Álvarez and Paula Esteller.*

In this practical part we will be analysing human RNA-seq expression data from the [GTEx consortium (version 8)](https://gtexportal.org/home/) using basic bash commands. For that we first need to download the [data](https://github.com/pesteller/Intro2Bioinfo_course/blob/main/GTEx_v8_gene_median_tpm.gct) from which we will be working on.

## 1.Exploring the dataset

Save the dataset in a bash variable:
```
GTEX=GTEx_v8_gene_median_tpm.gct
```

How does our dataset look like?
```
less -S $GTEX
```
![image](https://user-images.githubusercontent.com/68989675/178304599-195eb9b0-250f-4ac9-8f5b-5e19a735260c.png)


**Question 1.** Which is the difference between `less` and `less -S`?

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

<details closed>

  <summary>See answer 1</summary>

  ```bash
  grep "Name" $GTEX
  ```

</details>

<details><summary markdown="span">See answer 2</summary>
```bash
grep -v "ENS" $GTEX
```
</details>
<br/>



```
grep "Name" $GTEX
grep -v "ENS" $GTEX
```

**Q.** How many genes are in this dataset
```
tail -n +1 $GTEX | wc -l
```

Now we want to check how many tissues have been considered. We will do this with `awk`:

```
awk '{print NF}' $GTEX | head
```

You can see that the number of columns varies between rows. This is because there are spaces within IDs of the header (e.g. Adrenal Gland) 
Let's specify that the field separator between columns is a tab `\t` and check if this corrects the issue.
```
awk 'FS = "\t" {print NF}' $GTEX | head
```
This does not solve the problem, because `awk` cannot distinguish between spaces and tabs in this case.
For this we will use `sed` to remove the spaces within IDs:
``` 
head -1 $GTEX | sed 's/ //g' | awk '{FS = "\t"; print NF}'
```

But what if we want to do this change and keep it in the orginal file withtout creating a new one?
One could use `sed -i`; which directly modifies the input file. Careful: you need to be sure that the pattern you want to change is only present in your header so that it does not change the rest of the file.

In our case we first create a subfile:
```
sed  's/ //g' $GTEX > GTEX2
``` 

Then we use `diff` to check the differences between the original file and the subfile we have create as a sannity check:
```
diff $GTEX GTEX2
```

We can see how the only difference between the two files is in the header. Which confirms that we can use `sed -i` in the original file without messing up with the rest of the data:

```
sed -i 's/ //g' $GTEX
```

Let's check which is the column that has information about the "Liver".
There are several ways to do so:
* Option 1:
```
awk '{print $38}' $GTEX
```

* Option 2:
```
# First transpose the header line
head -1 $GTEX | tr "\t" "\n"
# Then we check with `grep` which is the position of "Liver"
head -1 $GTEX | tr "\t" "\n" | fgrep -n "Liver"
# And now we get the specific value and save it into a variable
col=$(head -1 $GTEX | tr "\t" "\n" | fgrep -n "Liver" | cut -d ":" -f1)
# This we can use in `awk`
 awk -v col=$col '{print $col}' $GTEX
```

* Option 2.1:
```
#Using the `$col` variable but with `cut`:
cut -f$col $GTEx | head
```

**Q** What is the average expression value in the Liver?
```
awk -v col=$col '{print $col}' $GTEX | tail -n 2 | awk '{sum+=$0} END {print sum/NR}'
```

Let's get some other metrics (min and max):
```
tail -n +2 $GTEX | cut -f $col | sort | tail
# We do not want this type of number sorting, so we add: -V or -g (depending on the version)
tail -n +2 $GTEX | cut -f $col | sort -V | tail
tail -n +2 $GTEX | cut -f $col | sort -g | tail
```

**Q.** What is the minimum? How many times is it present?
```
tail -n +2 $GTEX | cut -f $col | sort -g | uniq -c | sort -nr | head
```
