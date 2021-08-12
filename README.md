# Jobmigo - A Job / Talent recommendation platform
## Objective
We are a team of five, and we are going to work on a Job / Talent recommendation platform by match CV and Job posts. <br>

## Table of Contents
1.1 Web-scraping <br>
1.2 Data cleaning



# Job-Recommendation---1.2-Data-cleaning
## Objective
In our last repositories about web scraping, we talked about the websites store the information in a batch, and we have to split them into columns for our projects. In this page we will focus on data cleaning.

## Table of Contents
1. Columns that we want to split from web scraping & the Code by @hwhaa
2. Other data cleaning & code
3. Conclusions
4. Challenges

## 1. Columns that we want to split from what we scraped
* From [Heading Information] , we want: 
  * location
  * salary
  * post date
  ![data clean](image/data%20cleaning%2011.png)

* From [Additional Information] , we want:
  * Career Level
  * Qualification
  * Years of Experience
  * Job Type
  * Industry
  * Benefit
 ![data clean](image/data%20cleaning%2012.png)

* Besides, the employer didnt fill in all the information, so we also fill N/A to replace blank
## The Code
```
#just save job details
startTime = datetime.now()

job_title = []
co_name = []
location = []
salary = []
post_date = []
job_hl = []
job_d = []
co_overview = []
career_level=[]
qualification=[]
experience=[]
job_type=[]
industry = []
benefit = []
webpage=[]

counter=1
for i in range(8000):
    print(counter)
    html_d = requests.get(job_page[i])
    job = BeautifulSoup(html_d.text, "html.parser")
    
    #webpage
    webpage.append(job_page[i])
    
    #job title
    jt = job.h1.string
    job_title.append(jt)
    
    #co_name
    cn = job.find("div", {"data-automation":"detailsTitle"}).span.get_text()
    co_name.append(cn)
    
    #heading_info
    hi = [i.text for i in job.find_all(class_="sx2jih0 zcydq838")]
    
    #location
    location.append('N/A' if hi[0] == '' else hi[0])
    
    #salary
    salary.append(hi[1] if '$' in hi[1] else 'N/A')
    
    #post_date
    post_date.append(hi[-1])
    
    #job_highlights
    if job.find(class_="sx2jih0 sx2jih3 h6p8rp0 h6p8rp6"):
        jh = ", ".join([i.text for i in job.find(class_="sx2jih0 sx2jih3 h6p8rp0 h6p8rp6")])
        job_hl.append(jh)
    else:
        job_hl.append('N/A')
    
    #job_descriptions
    jd = job.find("div", {"data-automation":"jobDescription"}).get_text()
    job_d.append(jd)
    
    #company overview
    co=job.find_all(class_="sx2jih0 zcydq82b _18qlyvc0 _18qlyvcv _18qlyvc1 _18qlyvc8")[-1].text
    co_overview.append('N/A' if co==jd else co)
    
    #Additional info
    ai = [i.text for i in job.find_all(class_="sx2jih0 zcydq81w zcydq83k zcydq85t")]
    
    #career level
    for i in ai:
        if "Career Level" in i:
            career_level.append(re.search(r"Career Level(.+)",i).group(1))
        elif "Qualification" in i:
            qualification.append(re.search(r"Qualification(.+)",i).group(1))
        elif "Years of Experience" in i:
            experience.append(re.search(r"Years of Experience(.+)",i).group(1))
        elif "Job Type" in i:
            job_type.append(re.search(r"Job Type(.+)",i).group(1))
        elif "Industry" in i:
            industry.append(re.search(r"Industry(.+)",i).group(1))
        elif "Benefits & Others" in i:
            benefit.append(re.search(r"Benefits & Others(.+)",i).group(1))
    
    #fill N/A
    if len(career_level) != counter:
        career_level.append('N/A')
    if len(qualification) != counter:
        qualification.append('N/A')
    if len(experience) != counter:
        experience.append('N/A')
    if len(job_type) != counter:
        job_type.append('N/A')
    if len(industry) != counter:
        industry.append('N/A')
    if len(benefit) != counter:
        benefit.append('N/A')
    
    #put all the information into a dataframe
    jobs_all=pd.DataFrame({
    "title":job_title,
    "company":co_name, 
    "location":location,
    "salary":salary,
    "post_date":post_date,
    "job highlights":job_hl,
    "job description":job_d,
    "company overview":co_overview,
    "career level":career_level,
    "qualification":qualification,
    "experience":experience,
    "job type":job_type,
    "industry":industry,
    "benefit":benefit,
    "webpage":webpage})
    
    counter+=1


print(datetime.now() - startTime)
```
## 2. Other data cleaning & code
1. Salary - split salary range into 2 columns <br>
e.g. From [10,000-20,000] into [10,000] & [20,000]
```
#replace nan as 0 for calculation
dropped_j['salary'].fillna("0", inplace=True)

#create two columns as blank
dropped_j["sal_low"]= ""
dropped_j["sal_high"]=""

#cal Low & high
low = []
high = []
for i in dropped_j['salary']:
    x = i.replace(",","")
    x = re.findall('[0-9,]+', x)
    low.append(min(x))
    high.append(max(x))
#update the columns with low & high
new_df = pd.DataFrame({"sal_low": low, "sal_high":high})
dropped_j.update(new_df)
dropped_j
```

2. Fill NA as "Not Specified" in Career Level
in the later stage, we need the column ['Career Level'] as a filter on dashboard, so we have to replace N/A as "Not Specified"
```
dropped_j['career level'].fillna("Not Specified", inplace=True)
```

3. Combined columns and create 1 column as ['All'] for NLP process
```
#select columns and combined into 1 for NLP
cols = ['title', 'company', 'job highlights','job description','company overview', 'industry']
dropped_j['All'] = dropped_j[cols].apply(lambda row: ','.join(row.values.astype(str)), axis=1)
```

## 3. Conclusions
The data here is simple, we only have to put them into correct columns, fill NA
## 4. Challenges
Not standardised info <br>
from the employer, they dont provide all information, e.g. salary, location. So the information provided on the job page can vary depends on the information provided by the employer. After some searching on different job posts, we are able to find out all the hidden headlines. 
