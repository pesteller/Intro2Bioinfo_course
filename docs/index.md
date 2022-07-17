# Part I. Basics of bash
*Materials created by [Marina Álvarez](https://github.com/maralest) and [Paula Esteller](https://github.com/pesteller).*

In this practical part we will be analysing human RNA-seq expression data from the [GTEx consortium (version 8)](https://gtexportal.org/home/) using basic bash commands. For that we first need to download the [data](https://github.com/pesteller/Intro2Bioinfo_course/blob/main/GTEx_v8_gene_median_tpm.gct) from which we will be working on.

## 1. Exploring the dataset

Save the dataset in a bash variable:
```
GTEX=GTEx_v8_gene_median_tpm.gct
```

How does our dataset look like?
```
less -S $GTEX
```
![image](https://user-images.githubusercontent.com/68989675/178304599-195eb9b0-250f-4ac9-8f5b-5e19a735260c.png)


**Question 1.** What is the difference between `less` and `less -S`?

If you want to see it more nicely and column aligned you can use `column -t` option:
```
column -t $GTEX | less -S
```

You can also check which are the first or the last rows of your document by using `head $yourfile` or `tail $yourfile`.

**Question 2.** Save the header line of the document in 1 column. Save it in a new document entitled: `header.txt`
```
head -n 1 $GTEX | tr "\t" "\n" > header.txt
```
**Question 3.** Apart from using the `head` command, which other alternatives could you use to obtain the header?
```
grep "Name" $GTEX
grep -v "ENS" $GTEX
```

**Question 4.** How many genes are in this dataset
```
tail -n +2 $GTEX | wc -l
grep -c "ENS" $GTEX
```

Now we want to check how many tissues have been considered. We will do this with `awk`:

```
awk '{print NF}' $GTEX | head
```

You can see that the number of columns varies between rows. This is because there are spaces within IDs of the header (e.g. Adrenal Gland). Let's specify that the field separator between columns is a tab `\t` and check if this corrects the issue.
```
awk 'FS = "\t" {print NF}' $GTEX | head
```
This does not solve the problem, because `awk` cannot distinguish between spaces and tabs in this case.
For this we will use `sed` to remove the spaces within IDs:
``` 
head -1 $GTEX | sed 's/ //g' | awk '{FS = "\t"; print NF}'
```

But what if we want to apply this change in the orginal file withtout creating a new one?
One could use `sed -i`; which directly modifies the input file. Careful: you need to be sure that the pattern you want to change is only present in your header so that it does not change the rest of the file.

In our case, to make sure this is safe, we first create a subfile:
```
sed  's/ //g' $GTEX > GTEX2
``` 

And then we use `diff` to check the differences between the original file and the subfile we have created as a sannity check:
```
diff $GTEX GTEX2
```

We can see how the only difference between the two files is in the header. Which confirms that we can use `sed -i` in the original file without messing up with the rest of the data:

```
sed -i 's/ //g' $GTEX
```

**Question 5.** Apply another change to the file in order to change mid dashes (`-`) for underscores (`_`) and save the resulting file as `GTEx_v8_gene_median_tpm_formatted.gct`.

## 2. Retriving relevant information from the dataset

Let's check which is the column that has information about the *Liver*.
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

**Question 6.** What is the average expression value in the *Liver*?
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

**Question 7.** What is the minimum gene expressio value? How many times is it present?
```
tail -n +2 $GTEX | cut -f $col | sort -g | uniq -c | sort -nr | head
```

**Question 8.** Can you calculate the mean gene expression value for each tissue? Which of them has the highest mean expression value?
```
num_cols=$(head -1 $GTEX | tr "\t" "\n" | wc -l)

for i in $(seq 3 $num_cols);
do
   tissue=$(head -1 $GTEX | cut -f$i); 
   mean_expr=$(awk -v column=$i '{sum+=$column} END {print sum/NR}' <( tail -n +2 $GTEX)); 
   echo -e $tissue"\t"$mean_expr; 
done
```

**Question 9.** How many genes are lowly expressed (gene expression value < 0.1 TMP) in each tissue? Which of them has the highest number of lowly expressed genes?
```
num_cols=$(head -1 $GTEX | tr "\t" "\n" | wc -l)

for i in $(seq 3 $num_cols);
do
   tissue=$(head -1 $GTEX | cut -f$i); 
   low_expression=$(awk -v column=$i '{if ($column <= 0.1) {sum+=1}} END {print sum}' <( tail -n +2 $GTEX)); 
   echo -e $tissue"\t"$low_expression; 
done | sort -k2 -nr | head
```

**Question 10.** How many genes have a gene expression value < 0.1 TMP in each tissue? 

