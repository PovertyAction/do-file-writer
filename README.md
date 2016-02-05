DoFileWriter
============

`DoFileWriter` is a Mata class to write nicely formatted Stata do-files. It allows you to separate the contents and format of a do-file, delegating the formatting to `DoFileWriter`.

Without DoFileWriter
--------------------

You want to automate the creation of a do-file like this:

```
* Crosstab variable treatment against all other variables.
foreach var of varlist _all {
	if "`var'" == "treatment" ///
		continue
	tabulate `var' treatment
}
```

However, a Mata function to do so quickly becomes unwieldy with formatting concerns:

```
void write_do(string scalar filename)
{
	real scalar fh
	fh = fopen(filename, "w")
	fput(fh, "* Crosstab variable treatment against all other variables.")
	// No indent
	fput(fh, "foreach var of varlist _all {")
	// 1 indent
	fput(fh, char(9) + `"if "\`var'" == "treatment" ///"')
	// 2 indents
	fput(fh, 2 * char(9) + "continue")
	fput(fh, char(9) + "tabulate \`var' treatment")
	fput(fh, "}")
	fclose(fh)
}
```

Further, Mata requires the function to specify the file handle with each `fput()` call.

With DoFileWriter
-----------------

```
void write_do(string scalar filename)
{
	class `DoFileWriter' scalar df
	df.open(filename)
	df.put("* Crosstab variable treatment against all other variables.")
	df.put("foreach var of varlist _all {")
	// No need to specify indent
	df.put(`"if "\`var'" == "treatment" ///"')
	df.put("continue")
	df.put("tabulate \`var' treatment")
	df.put("}")
	df.close()
}
```

DIY
---

`DoFileWriter` analyzes a line to determine its appropriate indentation: this is its core functionality. However, you can turn off this behavior if you think `DoFileWriter` is going to get it wrong:

```
df.put("foreach var of varlist _all {")

// Turn off automatic indentation: lines inherit the indent of the line above.
df.set_autotab(`False')

// Indent the current line
df.indent()

// Indent as much as you like
df.indent(3)

// Decrease the indent of the current line
df.indent(-1)

// More decrease
df.indent(-2)

df.put(`"if "\`var'" == "treatment" ///"')
df.indent()
df.put("continue")
df.indent(-1)

// Auto-indentation back on
df.set_autotab(`True')

df.put("tabulate \`var' treatment")
df.put("}")
```

#delimit ;
----------

`DoFileWriter` automatically turns off auto-indentation when it encounters `#delimit ;`, as semicolon-delimited code tends to be more free-form. Auto-indentation returns to its previous setting at the next `#delimit cr`.

Installation
------------

`DoFileWriter` uses [Project Mata](https://github.com/PovertyAction/project-mata). Include it in your Project Mata project by adding it to `src` and updating `.matamac`.
