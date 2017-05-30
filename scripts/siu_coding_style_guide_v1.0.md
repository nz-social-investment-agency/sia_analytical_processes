# SIU Coding Style Guide
Author: Evidence and Insights  
Last updated: May 2017  


## Documentation
### Standardised Headers
All headers in scripts must use the standardised format. Examples of populated headers are shown below.

#### R header


```r
### ================================================================================================ ###
### Title: k10_sense_check.R
###
### Description: produce plots that compare the K10 pscyhological distress
### measure to the SF-36 perceieved disability measure
###
### Input: [IDI_Sandpit].[DL-MAA2016-15].[k10_vs_mh_band]
###
### Output: comparison_plot = large gg that produces a density plot
### of those with need and those with little or no need
### by average SF36 MH score
###
### Author: E Walsh
###
### Date: 24/2/2017
###
### Dependencies: k10_sense_check.sas builds the input table that is queried
### this can be run through main_sofie.sas
###
### SQL query located in sql/k10_vs_sf36_mh_check.sql
###
###
### Notes:
###
### Issues:
###
### History: 
### 24 Feb 2017 EW v1
### ================================================================================================ ###
```


#### SAS and SQL Header

```sas
/*********************************************************************************************************
TITLE: get_need_profile.sas

DESCRIPTION: Produce descriptive stats around those with an MHA need using the K10 score as
a proxy for need

INPUT:
sand.mha_pop_sofie_w7_wgt_adj

OUTPUT:
sf_sof_n_XXX = survey freq need cross tabulation output for variable XXX
sf_sof_n_XXX_rnd = rounded cross tabulation output for variable XXX

AUTHOR: E Walsh

DATE: 26 Jan 2017

DEPENDENCIES: see main_sofie.sas and run the libnames macros and formats scripts

NOTES: The sign out rules for SoFIE differ to the standard RR3 rounding. See pp.27-28 of the
SoFIE User Guide.

Issues:

HISTORY: 
26 Jan 2017 EW v1
*********************************************************************************************************/
```
### Inline comments
Comment should explain why you are doing things a particular way. If the code or concept is complex, then a few words explaining this concept or business rule is also required.

This is an example of a comment on what a piece of code is intended to do. Comments in the code must use the following notation for R.


```r
# Load aggregated treatment trends available in IDI Sandpit using the sql query file
```


This is an example of a comment on explaining the concept behind the code. **The `/* */` comments must be used in SAS, rather than just with a `*`** for ease of troubleshooting.


```sas
	/* Citalopram is used to treat both Mood-Anxiety and Dementia. Hence we can 
		definitively determine the diagnosis to be Mood anxiety only if there are 
		no other recorded cases of dementia for the individual.*/
```



While writing comments in SQL, do not use comments that start with double-hyphens (`--`). Always use `/* */` for comments in SQL scripts. This is because SAS wrappers are sometimes used to execute SQL scripts, and using a double-hyphen comment format may cause SAS wrapper code to fail.


## Naming Conventions
* Macro names, function names, file names (with the exception of the SIAL scripts) must be in lower case
* It is good practice to qualify dataset names with the library area to prevent any ambiguity,  e.g. `work.temp` 
* Temporary tables and columns use the prefix `_temp_`
* The script responsible for execution of the entire process flow must be called `main` and reside in `sasprogs` if it is a SAS script or `rprogs` if it is an R script (prefixes and suffixes are permitted as long as it is clear this is the main script).
* Use relative file-paths wherever possible. This makes it easier to move code to a different folder structure with minimal impact.

```sas
/* relative path */
%include "&si_source_path.\sasprogs\si_control.sas";
```

* Use consistent naming conventions for variables or columns, like _cnt, _cst, _dur, etc.
* If you are working on a version of the code do not name it after yourself. Use the original name and suffix with _v2, and add comments on the reason for the new version.
* Use lowercase and underscores to separate variable names. 
* **Do not** use camelCase. This makes it easier on the eyes.
* **Do not** include spaces in any file names or variable names. This minimises the need for escaping special characters and the chances that a single variable is mistaken for multiple variables.

## Data types
* It is generally preferred to use datetime rather than date, for better accuracy and consistency of code.

## Formatting of Code
* Arithmetic operators should be separated by white space.
* Code must be indented properly (*Hint:* Use Ctrl A + Ctrl I in SAS)
* Code lines generally should not exceed a length of 130 for purposes of readability
* One concept per line (see some examples below)



```r
# One line per pipe operator
domains <- as.data.frame(sofiecostdata  %>% 
                           dplyr::select(subject_area) %>% 
                           group_by(subject_area) %>% 
                           summarise() );
```


```sas
/* one line per clause */
  			proc sql noprint;
				select engine, sysvalue
					into: db_engine separated by ''
					, :db_schema separated by ''
				from dictionary.libnames
					where libname = "&db_lib";
			quit;
```

## Folder Structure
Most projects use the following file structures. The folder structure must be decided and implemented before a project is started.

* top_level_folder
      + docs (documentation to go in here)
      + examples (use this if you want a complete self contained example to be provided)
      + gitignore (anything we dont want to be seen in a public repo goes in here)
      + include (any code that you would like to reuse from other projects go in here)
      + logs (used for long SAS processes that won't fit in the GUI log)
      + output (summary statistics, model output etc)
        + plots (visual output)
      + rprogs (all R scripts go in here)
      + sasautos (all SAS macros go in here)
      + sasprogs (SAS scripts that are not macros go in here)
      + sql (all sql scripts go in here)
      
Note that some projects may not have R scripts. If you find the folders are not needed then delete them.

If you choose to completely deviate from this structure you need a good reason why. For example the SIAL project uses `bqa_complete` and `bqa_incomplete` to make it clear what scripts have had a business QA completed.

**Do not use EG projects to manage your scripts**

## Version Control
**Note** this section will require updating once the IDI has Gitlab installed.

* Versioning is a must. In the absence of a versioning tool use _v2 as a suffix on filenames, with ample comments in the history section of the code.
* **Do not** use your name as a suffix.
* Do not push code that does not run through to the repositories. In the future, hooks will prevent you from doing this.
* Tools will be made publicly available on the SIU github repository https://github.com/nz-social-investment-unit
* All major releases require an annotated tag.

```r
git tag -a v1.1.0 -m "v1.1.0"
```

Refer to our version control document to find out more about passing code through the firewall and how to clone repos to your pc.



## Programming practices
### Tasks prior to development
* An objective is required before coding can begin.
* It must be clear who the user is and what their requirements are. This will ensure fewer subsequent rewrites of the code.
* A design phase where process flows and other documentation are developed must be carried out prior to coding. This ensures that individual pieces of code fit together with each other. Note that the design phase may be revisited during the development process.


### Macros and functions
* Any piece of code that is likely to be reused must be in a SAS macro or an R function or a stored process.
* Functions must be in a separate script to the analysis script in R.
* SAS macros should be in separate files (one file per macro) and the filename should be the same as the macro name.
* SAS formats should also be kept in a separate file for ease of code maintenance.
* All SAS macro should be loaded into the library via sasautos.
* Where possible, do not define macros within macros. This tends to be inefficient and degrades code readability.


```sas
/* Load all the sofie macros automatically */
options obs=MAX mvarsize=max pagesize=132
        append=(sasautos=("&use_case_path\sasautos"));

```

### Improving efficiency
Run query plans so that the queries can be optimised. (Hit Ctrl + L when you are in the script window and find the large estimated cost percentages then see if the query can be rewritten in a better way).

Use explicit passthroughs where possible

```sas
	proc sql;
		connect to odbc (dsn=idi_clean_archive_srvprd);
		create table _temp_address_notification as
			select *
				from connection to odbc(
					select distinct
						a.snz_uid
						,b.ant_region_code
						,b.ant_ta_code
					from [IDI_Sandpit].[&si_char_proj_schema.].[&si_char_table_in.] a
						inner join [IDI_Clean].[data].[address_notification] b  
							on a.snz_uid = b.snz_uid and 
							a.&si_as_at_date between b.ant_notification_date and b.ant_replacement_date);
	quit;
```

Create clustered indexes on tables that are used often.

```sas
proc sql;
	connect to odbc(dsn=idi_clean_archive_srvprd);
	execute(create clustered index cluster_index on 
		[IDI_Sandpit].[&si_proj_schema].[&db_ds] (&si_index_cols)) by odbc;
	disconnect from odbc;
quit;
```

When joining large tables in SAS use hash objects. If the objects you have in SAS wont fit into memory then try using an index.

```sas
data &si_char_table_out. (drop = return_code:);
		set &si_char_table_in.;
/* dont declare the hash object each time a row is read */
		if _N_ = 1 then
			do;
				/* sneaky way to load the columns into the pdv without the data */
				if 0 then	set work._temp_personal_detail;
				declare hash hpd(dataset: 'work._temp_personal_detail');
				hpd.defineKey('snz_uid');
				hpd.defineData('snz_birth_year_nbr','snz_birth_month_nbr','as_at_age','snz_sex_code','snz_spine_ind');
				hpd.defineDone();
			end;	
					
		return_code_pd = hpd.find();

	run;
```

Turn on the SAS trace to see what the database is up to. Remember to disable this once code is production-ready or it will slow down the actual execution due to additional overhead of writing to log.

```sas
/* display the detail of the sql statements and calls */
/* the nostsuffix removes the trailer info that is difficult to read */
options sastrace=',,,d' sastraceloc=saslog nostsuffix;

```

Use mlogic to print out detailed SAS logs for ease of troubleshooting. Remember to disable this once code is production-ready.

```sas
/* display details of SAS statement execution */
options mlogic mprint;

```


### Error handling
For key programs try-catch statements are to be used in R


```r
r_data.frame <- function(conn, database_query){
			out <- tryCatch(
			  {
			    sqlQuery(channel = conn, query = paste(readLines(database_query), collapse="\n"))
			  },
			  error = function(err){
			    message(paste("Error in reading in data from db: ", err))
			  }
			)
			return(out)
}
```

For key programs in SAS use macro conditionals and note any errors or warnings in the log.


```sas
			%if "&db_engine" ~= "ODBC" %then
				%do;
					%put ERROR: In  md_write_to_db.sas - non ODBC engine specified. Are you sure you are writing to the database?;
				%end;
			%else
				%do;
          ...
        %end;
```

It is recommended to keep a debug provision while writing SAS macros which retains all temporary tables. this makes it easier to troubleshoot the code.

```sas
	/* clean up */
	%if %sysfunc(trim(&si_debug.)) = False %then
		%do;

			proc datasets lib=work;
				delete _temp_:;
			run;

		%end;
```

### Testing
#### Unit testing
* Macros/functions that are used often must be accompanied by unit tests
* Stress testing using large datasets should also be done to ensure the macro/function can cope with large amounts of data

The RUnit package will be used for R functions 

```r
test.<function_to_test> <- function() {
checkEquals(<function_to_test>(<function_parameters>), <returned_value>)
checkException(<function_to_test>("xx"))
```


Assert comments and macro calls are acceptable in SAS. Do not use the sasunit macros. This will create dependencies with another package that is not officially part of SAS that no one from within the team has QAd yet.


```sas

/* assert: should write a cluster index on two columns snz_uid and the date column */
%si_write_to_db(si_write_table_in=work.timedata, si_write_table_out=sand.test_cluster_index_two_col,
	si_cluster_index_flag=True, si_index_cols=%bquote(snz_uid, datetime));

/* checkException: should give you an error about positional parameters must precede keyword parameters */
%si_write_to_db(si_write_table_in=work.timedata, si_write_table_out 
	si_cluster_index_flag=True, si_index_cols=snz_uid, datetime);

/* assert: table &si_char_table_out. exists and is not empty */
/* assert: because this table is not in work you should also get a note about it being written to work */
/* stress test ~2.5 million rows run time ~ 1.5 minutes */
%si_get_characteristics(si_char_proj_schema=DL-MAA2016-15, si_char_table_in=distinct_mha_pop, 
	si_as_at_date=date_diagnosed, si_char_table_out=work.mha_pop_char);

```

#### Integration and user acceptance testing
* Integration testing is required. The main script must run from start to finish without an error.
* Someone who has not worked on developing the code must be able to figure out how to run the tool/analysis solely using the provided documentation.
* Testing needs to be done on at least two different environments to ensure that the code is robust.

### Temporary files
All temporary files in SAS should be deleted at the end of a script temporary files can be identified by the `_temp_` notation. It is recommended to wrap these statements around with a conditional execution based on a debug flag.

```sas
/* clean up */
proc datasets lib=work;
delete _temp_:
run;
```


## Graphics
### Choosing a Graphic
Think through the purpose of a graphic (whether it is exploratory, or part of the modelling process, or presentation of results) and the intended audience (you, other analysts, public) and hence choose graphics appropriately and only spend proportionate time on polishing.

* Test important graphics on a sample from the intended audience
* Graphics need to meet the audience’s need, and sometimes that means even a pie chart is right provided there aren't too many slices etc
* No redundant dimensions (eg 3d barcharts or piecharts)
* Non-traditional charts for specific purposes eg Sankey charts, mosaic plots, treemaps, diverging barcharts for likert scales can be useful in certain situations
* Ensure that graphics are simple and easily understandable.

### Visual Elements
* All plots must use the siu_theme
* Note that a contractor has built some SIU themed strictly increasing colour scales for us that can be used for ordered variables



```r
# example of using the SIU theme
sofie_wts_freqpoly <- 
  ggplot(sofie_wts_lng, aes(wt, colour = weight_var)) + 
  geom_freqpoly(bins = 50) +
  scale_colour_siu() + 
  theme_siu() +
  labs(x = "Weight", y = "Count", 
       title = "SoFIE Weight Distribution - Original vs. Adjusted") +
  scale_y_continuous(label = comma)
```

* Direct the eye to the data and its patterns, and minimise visual distraction from guides, labels and chartjunk.  Read Cleveland and Tufte.
* Gridlines should either be white (or pale grey) if on a grey or similar plotting area, or pale grey on a white (or near white) plotting area – always should be much less attention-grabbing than whatever represents the data.  No black or dark grey gridlines on white plotting area.
* Remove or at least minimise frames and borders around figures, plotting areas, legends; and if we must have them, make them pale grey rather than black.
* Put borders on tables into the visual background (eg- pale grey rather than black).
* Annotations, trend and smoothing lines, ellipses, etc all have their place if they draw the eye to the story.
* Minimal reliance on point shape and linetype – more than about three variants and they get hard to tell apart.


* Test important graphics through one of the online colour blind simulators
* Carefully choose between discrete, diverging (eg blue to grey to red) and strictly increasing (eg. pale blue to dark blue) scales.  Don’t use a discrete colour scale for an ordered variable (eg “year” in bar charts, better to go from pale blue to dark blue than have discrete colours)
* Discrete scales should (generally) have equal chroma and luminance
* Use well-designed and tested schemes like Viridis and Brewer when you can.
* In barcharts (only) the continuous value axis must go to zero; but in line charts (eg time series) and scatter plots axes limits should maximise the plotting area used for the data.
* Barcharts or dot plots should generally have the categories in some meaningful, non-alphabetical order (eg largest value to lowest), so just reading down the list of labels gives the reader something.
* Order levels in legends to minimise the eye’s workload in travelling from the legend to the data; and consider direct labels as an alternative.


### The text within a graphic
* Country names should follow MFAT or United Nations practice. 
* Clean up factor names etc before they go into the legend eg “Male” and “Female” rather than “SexMale”, “SexFemale”; and “New Plymouth” not “New_Plymouth”.

```r
... +
  scale_x_discrete(labels = c("Male", "Female"))
```

* Bar charts with multi-word labels for each bar will generally be more readable if the bars and text are horizontal


```r
... +
 coord_flip() 
```

* Use the correct fonts via the SIU theme, for text and label geoms in charts as well as for the basic elements.
* Don’t forget the macrons or other special characters eg Māori, whānau, Pākehā (in R, can use “\u0101” or equivalent instead of “a” to get an a with a macron).


```r
plot(1:10, 1:10, main = "M\u0101ori with a macron")
```

## Tabular output
It is recommended that tabular outputs conform to the pre-defined templates whenever possible. Use of these standardised templates enable StatsNZ to automate output checks and reduce the turnaround time taken for signing these out of the IDI.

* Model output such as mean estimates and their confidence intervals can be generated in the format specified by `../templates/model_output_stats.xlsx` and the function that builds this output is available in `../github_generic/rprogs`.
* Model output in the form of coefficients and p values may use the model_coefficients template. An example is shown in `../templates/model_coefficients.xlsx` and the function that builds this output can be found in `../github_generic/rprogs`.
* Clustering output may use the clustering_output template. An example is shown in `../templates/clustering_output.xlsx` and the function that builds this output can be found in `../github_generic/rprogs`.
* Numeric Summary Statistics should also conform to the defined template which can be found in `../templates/numeric_summary_stats.xlsx`.



**SIU Tracking Number: TBA**


Last updated June 2017 by Ernestynne Walsh and Vinay Benny
