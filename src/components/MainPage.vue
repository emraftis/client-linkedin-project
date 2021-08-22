<template>
  <div class="inputs">
    <label for="countryName">Country:</label>
    <select name="countryName" id="countryName" v-model="selectedCountryName" @change="getIndustryGrowth(selectedCountryName, selectedIndustryName)">
      <option v-for="country in countryNames" :key="country" :value="country">{{ country }}</option>
    </select>
    <br>
    <div v-if="selectedCountryName">
      <label for="industryName">Industry:</label>
      <select name="industryName" id="industryName" v-model="selectedIndustryName" @change="getIndustrySkills(selectedIndustryName)">
        <option v-for="industry in industryNames" :key="industry" :value="industry">{{ industry }}</option>
      </select>
    </div>
    <br>
    <button @click="getIndustrySkills(selectedIndustryName)">Search</button>
  </div>

  <div v-if="shouldDisplay" class="top-skills">
    <h3 v-if="topSkills.length > 0">{{ industryHeadingString }}</h3>
    <div class="list-container">
      <p>The Industry Skills Needs metric captures which skills are most likely to be added to a member's profile in one industry compared to other industries. It's calculated using an adapted version of a text mining technique called Term Frequency - Inverse Document Frequency (TF-IDF). This method gives more weight to a skill for an industry if more members in the industry list the skill on their profiles and the skill is more unique to the industry. The skills included are those added while a member holds a particular occupation (the skill flow approach). While the skill flow approach creates a trade-off whereby long-held basic skills, such as Microsoft Office being given a lesser weight, the approach is shown to be stronger at identifying the latest emerging skills in a specific industry than including all historical skills that are added during prior occupations. On balance, since the objective of this metric is to detect the latest skills needs, a skill flow approach is adopted.</p>
    </div>
    <div class="list-container">
      <ul v-if="topSkills.length > 0">
        <li v-for="skill in topSkills" :key="skill">{{skill}}</li>
      </ul>
    </div>
  </div>
  
  <div v-if="shouldDisplay" class="growth-rate">
    <h3>{{ growthHeadingString }}</h3>
    <div class="list-container">
      <p>The Growth from Industry Transitions metric looks at how LinkedIn members are moving across industries (based on net transition: in minus out). For example, if a net of 50 members joined a Biotech industry with 1,000 members in a particular country after leaving jobs in other industries, that Biotech industry would have grown by 5% due to transitions in that year. We calculate these rates across all industries on an annual basis, and report an average of the last three years. The metric above is built entirely on a sample of LinkedIn members that have a company registered on LinkedIn on their profile. Since white-collar workers in knowledge-intensive services sectors are more likely to be on LinkedIn, the growth rate above may not accurately represent sectors like manufacturing and mining that tend to have more blue-collar workers.</p>
    </div>
    <div class="chart-container" v-if="industryGrowth">
      <area-chart width="100%" suffix="%" xtitle="Year" ytitle="%" :data='{
        2015:industryGrowth[0].rate, 
        2016:industryGrowth[1].rate, 
        2017:industryGrowth[2].rate, 
        2018:industryGrowth[3].rate, 
        2019:industryGrowth[4].rate
        }'
      ></area-chart>
    </div>
    <div v-if="industryGrowth" class="list-container">
      <caption class="caption-text"><a href="https://documents1.worldbank.org/curated/en/827991542143093021/pdf/World-Bank-Group-LinkedIn-Data-Insights-Jobs-Skills-and-Migration-Trends-Methodology-and-Validation-Results.pdf">"World Bank LinkedIn Digital Data for Development" by World Bank Group & LinkedIn Corporation, licensed under CC BY 3.0.</a></caption>
    </div>
    <div v-else>
      <h4>No Data Available</h4>
    </div>
  </div>
</template>

<script>
const axios = require('axios');

export default {
  name: 'IndustrySkillsComponent',
  components: {
  },
  data() {
    return {
      industryNames: ["Mining & Metals",
        "Oil & Energy",
        "Pharmaceuticals",
        "Food Production",
        "Aviation & Aerospace",
        "Automotive",
        "Chemicals",
        "Machinery",
        "Textiles",
        "Paper & Forest Products",
        "Printing",
        "Electrical & Electronic Manufacturing",
        "Plastics",
        "Renewables & Environment",
        "Packaging & Containers",
        "Industrial Automation",
        "Computer Hardware",
        "Computer Software",
        "Computer Networking",
        "Internet",
        "Semiconductors",
        "Telecommunications",
        "Motion Pictures & Film",
        "Broadcast Media",
        "Newspapers",
        "Publishing",
        "Information Technology & Services",
        "Writing & Editing",
        "Computer Games",
        "Online Media",
        "Computer & Network Security",
        "Media Production",
        "Banking",
        "Insurance",
        "Financial Services",
        "Investment Banking",
        "Investment Management",
        "Venture Capital & Private Equity",
        "Law Practice",
        "Legal Services",
        "Management Consulting",
        "Biotechnology",
        "Veterinary",
        "Accounting",
        "Architecture & Planning",
        "Research",
        "Executive Office",
        "Marketing & Advertising",
        "Information Services",
        "Environmental Services",
        "Market Research",
        "Public Relations & Communications",
        "Design",
        "Professional Training & Coaching",
        "Translation & Localization",
        "Events Services",
        "Outsourcing/Offshoring",
        "Mechanical Or Industrial Engineering",
        "Photography",
        "Graphic Design",
        "Entertainment",
        "Gambling & Casinos",
        "Sports",
        "Museums & Institutions",
        "Fine Art",
        "Performing Arts",
        "Arts & Crafts",
        "Music",
        "Health, Wellness & Fitness",
        "Animation"],
      countryNames: ["Australia","Brazil","Canada","Costa Rica","Denmark","France",
      "Georgia","Germany","Greece","Hong Kong SAR, China","Ireland","Italy", "Japan","Korea, Rep.","Lebanon",
      "Mexico","Malaysia","New Zealand","United Kingdom","Philippines","Puerto Rico",
      "Portugal","Singapore","South Africa","Spain","Switzerland","Thailand","United States","Vietnam"],
      selectedIndustryName: '',
      selectedCountryName: '',
      industryAPIString: '',
      countryAPIString: '',
      industryHeadingString: '',
      growthHeadingString: '',
      topSkills: [],
      industryGrowth: [],
      isLoading: false,
    }
  },
  computed: {
    shouldDisplay() {
      if (this.isLoading || this.topSkills.length === 0) {
        return false
      }
      return true
    },
  },
  methods: {
    async getIndustrySkills(industry) {
      this.isLoading = true
      this.selectedIndustryName = industry;
      if (industry.includes(" & ")) {
      this.industryAPIString = industry.replace(' & ', '%20%26%20')
      }
      else if (industry.includes(" ")) {
      this.industryAPIString = industry.replace(' ', '%20')
      } else {
        this.industryAPIString = industry;
      }

      if (this.industryAPIString && this.countryAPIString) {
        this.getIndustryGrowth(this.countryAPIString, this.industryAPIString)
      }

      try {
        const res = await axios.get(`api/top-skills?ind=${this.industryAPIString}`)
        const data = res.data;
        this.topSkills = data;
        this.industryHeadingString = "Top Skill Groups for " + this.selectedIndustryName + ":";
        this.isLoading = false;
        return;
      } catch (e) {
        console.log(e);
        return e;
      }
    },
    async getIndustryGrowth(country, industry) {
      this.isLoading = true;
      if (!country || industry === null) {
        this.isLoading = false;
        return;
      }
      this.selectedCountryName = country;
      this.countryAPIString = country.toString();
      if (this.countryAPIString.includes(",")) {
        this.countryAPIString = industry.replace(',', '%2C')
        if (this.countryAPIString.includes(" ")) {
          this.countryAPIString = this.countryAPIString.replace(' ', '%20')
          if (this.countryAPIString.includes(".")) {
            this.countryAPIString = this.countryAPIString.replace('.', '%2E')
            if (this.countryAPIString.includes("'")) {
              this.countryAPIString = this.countryAPIString.replace("'", '%27')
            }
          }
        }
      } this.countryAPIString = country;
      try {
        const res = await axios.get(`api/ind-growth?industry=${this.industryAPIString}&country=${this.countryAPIString}`)
        const data = res.data;
        if (!data) {
          this.industryGrowth = null;
        } if (data) {
          this.industryGrowth = [{year: 2015, rate: data.growth_rate_2015}, {year: 2016, rate: data.growth_rate_2016}, {year: 2017, rate: data.growth_rate_2017}, {year: 2018, rate: data.growth_rate_2018}, {year: 2019, rate: data.growth_rate_2019}]
        }
        this.growthHeadingString = "Growth for " + this.selectedIndustryName + " in " + this.selectedCountryName;
        this.isLoading = false;
        return;
      } catch (e) {
        console.log(e);
        return e;
      }
    }
  },
}
</script>

<style scoped>
body {
  margin: 0;
  padding: 0;
}
h2 {
  margin-bottom: 0;
  margin-top: 2%;
}
ul {
  padding: 0;
}
li {
  margin: 0 10px;
  display: flex;
}
.list-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50%
}
.chart-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 95%;
}
a {
  color: #42b983;
}
p {
  display: flex;
  font-size: 0.85rem;
}
#countryName,
#industryName {
  padding: 0.15rem;
  margin: 0.3rem
}
.caption-text {
  font-size: 0.7rem;
}
@media screen and (min-width: 850px) {
  .chart-container {
    display: flex;
    width: 40%;
  }

  canvas {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .growth-rate, .top-skills {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}
</style>
