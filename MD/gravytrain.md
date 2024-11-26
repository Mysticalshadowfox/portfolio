MP expenses are a *hot* topic in the UK, and the local MP raised eyebrows  with large claims for rented accommodation, even though they live in a constituency close to London. This made me wonder if the outrage was justified.
                  
The outcome? The Gravy Train. Now armed with data, the question is: How does your MP's spending compare?

<a href="https://gravytrain.streamlit.app" target="_blank" class="center-align"><img src="images/GravyTrain_Screenshot.png" ></a>

<a href="https://gravytrain.streamlit.app" target="_blank"><p>https://gravytrain.streamlit.app</p></a>

---

<h3 class="light-blue-text darken-1">The Gravy Train Data Stack</h3>

<<img src="images/EtL.png" class="">

---

<h3 class="light-blue-text darken-1">The Gravy Train Pipeline</h3>

#### Data Sources
**UK Parliament**

The [UK Parliament](https://www.parliament.uk/) website hosts [Resources](https://developer.parliament.uk) for developers, including a directory of open data APIs which allow Parliamentary data to be shared publicly. All data is made available under the Open Parliament Licence. The following data is extracted.

<li>MP data.</li>
<li>MP historic name data.</li>
<li>Constituency details.</li>
<li>Constituency historic representation data.</li>
<li>Constituency geometry.</li>

**IPSA**

The [Independent Parliamentary Standards Authority](https://www.theipsa.org.uk/), the independent body that regulates and administers the business costs and decides the pay and pensions of the 650 elected MPs and their staff in the UK. [Expense](https://www.theipsa.org.uk/mp-staffing-business-costs/annual-publications) claim data is released and updated every two months and includes details of all publishable claims for all MPs for a financial year. The following data is extracted. 

<li>All published expense claims 2010 - 2023, approx 2 million** records.</li>


** *See update*

#### Extract transform Load

In the process of extracting data from these sources, various ETL options were explored. After experimenting with several open-source ETL offerings, a bespoke, parameter-driven ETL process named the [**"Quack Stack EtL Pipeline"**](https://github.com/JasonMuteham/Quack_Stack_EtL_Pipeline) was developed. The code is available on [GitHub](https://github.com/JasonMuteham/Quack_Stack_EtL_Pipeline). The data is initially extracted from sources and stored locally as CSV files, which are then loaded into a database, either [DuckDB](https://duckdb.org/) or [MotherDuck](https://motherduck.com/) using DuckDB's native CSV import functions. While it is possible to load data similarly into MotherDuck, reliability issues were encountered as MotherDuck is still in development. The most dependable approach involved creating a local DuckDB database and utilizing the MotherDuck extension to replicate the database in the cloud.


#### DBT

The Data Build Tool from [**DBT Labs**](https://www.getdbt.com/) is used to transform the raw data into tables and views.  

**Geojson**

Constituency geometry data undergoes transformation using the DuckDB spatial extension, calculating additional geometry data and exporting the results as a geojson file. This file serves as a layer when generating the app map. Notably, MotherDuck does not currently support the spatial extension.

#### Powering Streamlit with Data

Various methods are available to supply the [Streamlit](https://streamlit.io/) app with data, the following were considered.

<li> Bundle the DuckDB database within the app.</li>
<li> Direct connection to a cloud database with MotherDuck.</li>
<li> Write an API to serve the data to the app.</li>


All options have been tested and are performant. Ultimately the app is packaged with parquet files that are queried in app by DuckDB. Why? well.


<li> Free hosting services have a file size limit less that the DuckDB database size eg. GitHub 100mb.</li>
<li>MotherDuck is still developmental.</li>
<li>The size of the parquet files required are compact (<50mb) and can be queried directly as tables with DuckDB.</li>
<li>DuckDB is fast.</li> 
<li>The underlying data changes slowly.</li>


#### Streamlit

The [Streamlit](https://streamlit.io/) app loads constituency geometry via the geojson file and conducts data queries on the parquet files. The merged data is then passed to [pydeck](https://deckgl.readthedocs.io/en/latest/), where it is displayed as an interactive map layer.

#### Update

A few weeks after releasing the [Gravy Train](https://gravytrain.streamlit.app) app the IPSO stopped publishing individual transactions for “MP Travel” and removed over 1 million records from the dataset. The missing data is crucial for the analysis. The IPSO does produce an aggregate dataset based on the financial year. The pipelines and app have been updated to use the alternative data source. A few observations on the aggregate dataset.

<li>The format of the data is inconsistent between financial years.</li>
<li>Aggregate totals do not match the individual transactions.</li>
<li>Data for a financial year is release & updated many months, years after the period has ended.</li>


All welcome challenges to exercise the data engineering skill set! The new dataset also provided an opportunity to try out a different data orchestration framework in this case [Mage](https://www.mage.ai/). 




