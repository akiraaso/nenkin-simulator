<template>
  <div>
    <div id="container">
      <aside id="banner">
        <a id="github-link" href="#">
          <h1 id="banner-title">ねんきんシミュレーター</h1>
          &nbsp;
          <span>hosted with <span style="color: #e25555;">&hearts;</span> by <strong>GitHub</strong></span>
        </a>
      </aside>
      <aside id="sidebar">
        <div id="variables-pane">
          <div>
            <label for="">月々の積立金額(円):</label>
            <span>{{ monthlyPayment }}</span>
            <input
              v-model.number="monthlyPayment"
              type="range"
              :min="this.minMonthlyPayment"
              :max="this.maxMonthlyPayment"
              step="1000"/>
          </div>
          <div>
              <label>年利回り(%):</label>
              <span>{{ (apy * 100).toFixed(1) }}</span>
              <input
                v-model.number="apy"
                type="range"
                :min="this.minApy"
                :max="this.maxApy"
                step="0.001"/>
          </div>
          <div>
            <label>受給開始年(歳):</label>
            <span>{{ yearReceiptBegins }}</span>
            <input
              v-model.number="yearReceiptBegins"
              type="range"
              min="60"
              max="75"
              step="1"/>
          </div>
        </div>
        <ul id="chart-list">
          <li class=chart-list-item v-for="chartData in chartDatum.filter((d) => d.type === 'employeesPension')" :key="chartData.name">
            <span class="chart-list-label" :style="{ backgroundColor: chartData.color }"></span>
            <span>厚生年金(等級{{chartData.grade}})</span>
          </li>
          <li class=chart-list-item v-for="chartData in chartDatum.filter((d) => d.type === 'nationalPension')" :key="chartData.name">
            <span class="chart-list-label" :style="{ backgroundColor: chartData.color }"></span>
            <span>国民年金({{nationalOptionNames.filter((d) => d.value === chartData.option)[0].name}})</span>
          </li>
          <li class=chart-list-item v-for="chartData in chartDatum.filter((d) => d.type === 'noPension')" :key="chartData.name">
            <span class="chart-list-label" :style="{ backgroundColor: chartData.color }"></span>
            <span>年金なし</span>
          </li>
        </ul>
      </aside>
      <div id="chartPanel" ref="chartPanel" >
        <div id="chartWrapper">
          <svg id="chart" ref="chart"></svg>
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>
/* Container */

#container {
  --main-bg-color: #f0f0f0;
  --font-black: #2c2c2c;
  --shadow-color: rgba(60, 63, 66, 0.32);
  --base-margin: 5vw;

  /* 0px is converted to 0, it silently causes an error, hence using 0.01px */
  /* https://stackoverflow.com/questions/62520998/why-doesnt-min-or-max-work-with-unitless-0 */
  padding: 0 max(calc((100vw - 1700px) / 2), 0.01px);

  display: grid;
  height: 100vh;
  grid-template-rows: 1fr auto;
  grid-template-columns: auto 1fr;
  grid-template-areas:
    "sidebar chartPanel"
    "sidebar banner"
  ;
  position: relative;
}

/* main background */
#container::before {
  background-color: var(--main-bg-color);
  background-image:
    radial-gradient(at 23% 88%, rgba(153, 99, 0) 0px, transparent 80%),
    radial-gradient(at 13% 14%, rgba(0, 153, 122) 0px, transparent 80%),
    radial-gradient(at 40% 65%, rgba(5, 130, 188) 0px, transparent 50%),
    radial-gradient(at 70% 8%, hsla(310,100%,30%,1) 0px, transparent 80%),
    radial-gradient(at 96% 92%, hsla(229,100%,30%,1) 0px, transparent 80%);
  opacity: 0.08;
  background-position: 50% 50%;
  width: 100%;
  height: 100%;
  content: '';
  position: absolute;
  z-index: -1;
  background-size: cover;
}

a {
  color: inherit;
  text-decoration: inherit;
}

h1 {
  font: inherit;
  margin: unset;
}

* {
  box-sizing: border-box;
  font-family: sans-serif;
}

/* Banner */

#banner {
  grid-area: banner;
  display: flex;
  align-items: flex-start;
  justify-content: right;
  font-size: 0.8rem;
  height: max(var(--base-margin), 2rem);
  padding: 1rem calc(var(--base-margin) + 1rem) 0 0;
}

#github-link {
  color: #444;
  opacity: 0.8;
}

#github-link:hover {
  text-decoration: underline;
}

#banner-title {
  display: inline;
}

/* Sidebar */

#sidebar {
  grid-area: sidebar;
  color: var(--font-black);
  width: 18rem;
  height: 100vh;
  padding: var(--base-margin) 0;
  margin: 0 2vw 0 var(--base-margin);
  display: grid;
  grid-template-areas: "variables-pane" "chart-list";
  grid-template-rows: auto minmax(0, 1fr);
}

/* Variables Pane */

#variables-pane {
  grid-area: variables-pane;
  font-size: 0.9rem;
  margin: 0.25rem 0 calc(var(--base-margin) / 2);
  border-radius: 1rem;
  box-shadow: var(--convex-shadow);
}

#variables-pane input {
  display: block;
  margin: 5px 0 10px;
}

#variables-pane div:last-of-type > input {
  margin-bottom: 0;
}

/* Chart List */

#chart-list {
  grid-area: chart-list;
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  margin: 0;
  padding: 0;
  border-radius: 1rem;
  box-shadow: var(--concave-shadow);
}

.chart-list-item {
  list-style-type: none;
  display: block;
  font-size: 0.8rem;
  padding: 0.25rem 0.25rem;
  height: 1.2rem;
}

.chart-list-item-title {
  display: flex;
  align-items: center;
}

.chart-list-label {
  display: inline-block;
  height: 0.5rem;
  width: 0.5rem;
  border-radius: 1rem;
  margin-right: 0.25rem;
}

/* Chart panel */

#chartPanel {
  grid-area: chartPanel;
  align-self: stretch;
  align-items: center;
  border-radius: 1rem;
  box-shadow: 2px 2px 10px var(--shadow-color);
  background-color: white;
  display: flex;
  margin: var(--base-margin) var(--base-margin) 0 0;
}

#chartWrapper {
  width: calc(100% - 2rem);
  margin: 0 auto;
  display: flex;
  justify-content: center;
}

#chart {
  width: min(100%, 1000px);
  overflow: visible;
}

@media all and (max-width: 1024px) {
  #container {
    height: auto;
    grid-template-rows: auto auto minmax(0, 1fr);
    grid-template-columns: auto;
    grid-template-areas:
      "banner"
      "chartPanel"
      "sidebar"
    ;
  }

  #banner {
    font-size: 0.7rem;
    align-items: flex-end;
    padding: 0 calc(3vw + 1rem) 1.5vw 0;
  }

  #sidebar {
    height: 70vh;
  }

  #variables-pane {
    font-size: 0.8rem;
  }

  #chartPanel {
    margin: 0 3vw 0 3vw;
  }

  .chart-list-item {
    font-size: 0.7rem;
  }
}
</style>

<script lang="ts">
import Vue from 'vue';
import * as d3 from 'd3';
import employeesPensionContributionCsv from '~/assets/employees-pension-contribution.csv';

type nationalPensionOption = 'standard' | 'additive' | 'exemptFull' | 'exempt34' | 'exempt12' | 'exempt14';
type pensionType = 'noPension' | 'nationalPension' | 'employeesPension';
interface employeesPensionGrade {
  grade: string,
  standard_income: string,
  income_least: string,
  income_under: string,
  full_contribution: string,
  employee_contribution: string,
}
interface _chartDataProto {
  name: string;
  type: pensionType;
  apy: number;
  monthlyPayment: number;
  yearInvestmentEnds: number;
  color: string;
  data: number[] | null;
  error: Error | null;
}
interface noPensionData extends _chartDataProto {
  type: 'noPension';
}
interface nationalPensionData extends _chartDataProto {
  type: 'nationalPension';
  yearReceiptBegins: number;
  option: nationalPensionOption;
}
interface employeesPensionData extends _chartDataProto {
  type: 'employeesPension';
  yearReceiptBegins: number;
  grade: number;
  includeNational: boolean;
}
type chartData = noPensionData | nationalPensionData | employeesPensionData;
type valued<T extends chartData> = Omit<T, 'data'> & { data: number[]; }

const EMPLOYEES_PENSION_GRADES = d3.csvParse<keyof employeesPensionGrade>(employeesPensionContributionCsv);

const MAX_YEAR_LIMIT: number = 100;
const MIN_YEAR_LIMIT: number = 1;
const MAX_APY: number = 0.2;
const MIN_APY: number = 0;

function exhaustiveCheck(neverParam: never, message: string): never {
  throw new Error(message);
}

function subCmap(
  cmap: (arg: number) => string,
  vmin: number = 0,
  vmax: number = 1,
  reverse: boolean = false,
): (arg: number) => string {
  if (
    vmax < vmin ||
    vmin < 0 ||
    vmin > 1 ||
    vmax < 0 ||
    vmax > 1
  ) {
    console.error(`Unexpected vmin ${vmin} and/or vmax ${vmax}.`);
    return cmap;
  }
  return (v: number) => {
    if (reverse) {
      return cmap(vmax - (vmax - vmin) * v);
    }
    return cmap(vmin + (vmax - vmin) * v);
  };
}

/**
 * National Pension's contribution and benefit for a month
 * https://www.nenkin.go.jp/service/kokunen/hokenryo/20150331.html
 * https://www.nenkin.go.jp/service/kokunen/hokenryo/20150331-03.html
 * https://www.nenkin.go.jp/oshirase/taisetu/2021/202104/202104nenkingaku.html
 * https://ja.wikipedia.org/wiki/%E5%9B%BD%E6%B0%91%E5%B9%B4%E9%87%91
 *
 * @returns {[number, number]} an array of contribution and benefit of national pension for a month
 */
function monthlyNationalCB(option: nationalPensionOption, yearReceiptBegins: number): [number, number] {
  const standardContribution = 16610;
  let monthlyContribution;
  let monthlyBenefit;
  let benefitRate: number;
  switch (option) {
    case 'standard':
      monthlyContribution = standardContribution;
      monthlyBenefit = 780900 * rateByDelay(yearReceiptBegins) / 12;
      break;
    case 'additive':
      monthlyContribution = standardContribution + 400;
      monthlyBenefit = (780900 + 96000) * rateByDelay(yearReceiptBegins) / 12;
      break;
    case 'exemptFull':
      benefitRate = 1 / 2;
      monthlyContribution = 0
      monthlyBenefit = 780900 * rateByDelay(yearReceiptBegins) / 12 * benefitRate;
      break;
    case 'exempt34':
      benefitRate = 5 / 8;
      monthlyContribution = standardContribution * 1 / 4;
      monthlyBenefit = 780900 * rateByDelay(yearReceiptBegins) / 12 * benefitRate;
      break;
    case 'exempt12':
      benefitRate = 6 / 8;
      monthlyContribution = standardContribution * 1 / 2;
      monthlyBenefit = 780900 * rateByDelay(yearReceiptBegins) / 12 * benefitRate;
      break;
    case 'exempt14':
      benefitRate = 7 / 8;
      monthlyContribution = standardContribution * 3 / 4;
      monthlyBenefit = 780900 * rateByDelay(yearReceiptBegins) / 12 * benefitRate;
      break;
    default:
      monthlyContribution = 0;
      monthlyBenefit = 0;
      console.error(`Unexpected value ${option} in monthlyNationalCB`);
  }
  return [monthlyContribution, monthlyBenefit];
}

/**
 * @param yearInvestmentEnds year in which invenstment ends
 * @param yearReceiptBegins year in which pension receipt begins
 * @returns array whose index is year, value is balance, and length is `yearInvestmentEnds`
 */
function noPension({
  apy,
  monthlyPayment,
  yearInvestmentEnds,
}: Readonly<noPensionData>
): number[] {
  const years = d3.range(1, yearInvestmentEnds + 1);
  const yearlyRate = 1 + apy;
  const monthlyRate = yearlyRate ** (1 / 12);
  const investmentSumPerYear = geometricSeries(monthlyPayment, monthlyRate, 12);
  return years.map((year) => geometricSeries(investmentSumPerYear, yearlyRate, year));
}

/**
 * @param yearInvestmentEnds year in which invenstment ends
 * @param yearReceiptBegins year in which pension receipt begins
 * @returns array whose index is year, value is balance, and length is `yearInvestmentEnds`
 */
function nationalPension({
  apy,
  monthlyPayment,
  yearInvestmentEnds,
  yearReceiptBegins,
  option,
}: Readonly<nationalPensionData>
): number[] {
  const [monthlyContribution, monthlyBenefit] = monthlyNationalCB(option, yearReceiptBegins);
  const monthlySurplus = monthlyPayment - monthlyContribution;

  // Error check
  verifyApy(apy);
  verifyYearInvestmentEnds(yearInvestmentEnds);
  verifyYearReceiptBegins(yearReceiptBegins);
  if (monthlySurplus < 0) {
    throw new RangeError(`MonthlyPayment must be greater than or equal to ${monthlyContribution} (monthlyContribution).`);
  };

  const YearContributionEnds = d3.min([yearInvestmentEnds, 40]);
  if (YearContributionEnds === undefined) {
    throw new Error("d3.min failed for YearContributionEnds");
  }
  const yearlyRate = 1 + apy;
  const monthlyRate = yearlyRate ** (1 / 12);
  const surplusSumPerYear = geometricSeries(monthlySurplus, monthlyRate, 12);
  const pensionSumPerYear = geometricSeries(monthlyBenefit, monthlyRate, 12);
  const investmentSumPerYear = geometricSeries(monthlyPayment, monthlyRate, 12);
  const balances = Array(yearInvestmentEnds);

  // Populate balances
  balances[0] = surplusSumPerYear;
  for (const i of d3.range(1, YearContributionEnds)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + surplusSumPerYear;
  }
  for (const i of d3.range(40, yearReceiptBegins - 20 - 1)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + investmentSumPerYear;
  }
  for (const i of d3.range(yearReceiptBegins - 20 - 1, yearInvestmentEnds)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + investmentSumPerYear + pensionSumPerYear;
  }

  return balances;
}

/**
 * @param yearInvestmentEnds year in which invenstment ends
 * @param yearReceiptBegins year in which pension receipt begins
 * @param includeNational whether or not pension benefit includes national pension's
 * @returns array whose index is year, value is balance, and length is `yearInvestmentEnds`
 */
function employeesPension({
  apy,
  monthlyPayment,
  yearInvestmentEnds,
  yearReceiptBegins,
  grade,
  includeNational = true,
}: Readonly<employeesPensionData>
): number[] {
  verifyApy(apy);
  verifyYearInvestmentEnds(yearInvestmentEnds);
  verifyYearReceiptBegins(yearReceiptBegins);

  const gradeInfo = EMPLOYEES_PENSION_GRADES[grade - 1];
  if (gradeInfo === undefined) {
    throw new ReferenceError(`employeesPensionGrade not found for grade ${grade}`);
  }
  const standardIncome = gradeInfo.standard_income;
  if (standardIncome === undefined) {
    throw new ReferenceError(`standardIncome not found for grade ${grade}`);
  }
  const contributionStr = gradeInfo.employee_contribution;
  if (contributionStr === undefined) {
    throw new ReferenceError(`contributionStr not found for grade ${grade}`);
  }
  const monthlyContribution = Number(contributionStr);
  // https://www.nenkin.go.jp/service/yougo/hagyo/hoshuhirei.html
  const standardYearlyBenefit = Number(standardIncome) * 5.481 / 1000 * 480;
  const monthlyBenefit = (standardYearlyBenefit + (includeNational ? 780900 : 0)) * rateByDelay(yearReceiptBegins) / 12;
  const monthlySurplus = monthlyPayment - monthlyContribution;
  if (monthlySurplus < 0) {
    throw new RangeError(`MonthlyPayment must be greater than or equal to ${monthlyContribution} (monthlyContribution) for grade ${grade}.`);
  };

  const YearContributionEnds = d3.min([yearInvestmentEnds, 40]);
  if (YearContributionEnds === undefined) {
    throw new Error("YearContributionEnds can not calculated.");
  }
  const yearlyRate = 1 + apy;
  const monthlyRate = yearlyRate ** (1 /12);
  const surplusSumPerYear = geometricSeries(monthlySurplus, monthlyRate, 12);
  const pensionSumPerYear = geometricSeries(monthlyBenefit, monthlyRate, 12);
  const investmentSumPerYear = geometricSeries(monthlyPayment, monthlyRate, 12);
  const balances = Array(yearInvestmentEnds);

  // Populate balances
  balances[0] = surplusSumPerYear;
  // Period when pension contribution paid
  for (const i of d3.range(1, YearContributionEnds)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + surplusSumPerYear;
  }
  // Period when pension contribution nor pension benefit paid
  for (const i of d3.range(40, yearReceiptBegins - 20 - 1)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + investmentSumPerYear;
  }
  // Period when pension benefit paid
  for (const i of d3.range(yearReceiptBegins - 20 - 1, yearInvestmentEnds)) {
    const lastBalance = balances[i - 1];
    balances[i] = lastBalance * yearlyRate + investmentSumPerYear + pensionSumPerYear;
  }

  return balances;
}

function verifyApy(apy: number): void {
  if (MAX_APY == null) {
    throw new ReferenceError('MAX_APY is not set.');
  }
  if (MIN_APY == null) {
    throw new ReferenceError('MIN_APY is not set.');
  }
  if (apy > MAX_APY) {
    throw new RangeError(`apy ${apy} is greater than MAX_APY ${MAX_APY}`);
  }
  if (apy < MIN_APY) {
    throw new RangeError(`apy ${apy} is less than MIN_APY ${MIN_APY}`);
  }
}

function verifyYearInvestmentEnds(yearReceiptBegins: number): void {
  if (MAX_YEAR_LIMIT == null) {
    throw new ReferenceError('MAX_YEAR_LIMIT is not set.');
  }
  if (MIN_YEAR_LIMIT == null) {
    throw new ReferenceError('MIN_YEAR_LIMIT is not set.');
  }
  if (yearReceiptBegins > MAX_YEAR_LIMIT) {
    throw new RangeError(`yearReceiptBegins ${yearReceiptBegins} is greater than MAX_YEAR_LIMIT ${MAX_YEAR_LIMIT}`);
  }
  if (yearReceiptBegins < MIN_YEAR_LIMIT) {
    throw new RangeError(`yearReceiptBegins ${yearReceiptBegins} is less than MIN_YEAR_LIMIT ${MIN_YEAR_LIMIT}`);
  }
}

function verifyYearReceiptBegins(yearReceiptBegins: number): void {
  if (yearReceiptBegins < 60) {
    throw new RangeError(`yearReceiptBegins ${yearReceiptBegins} must be greater than or equal to 60`);
  }
  if (yearReceiptBegins > 75) {
    throw new RangeError(`yearReceiptBegins ${yearReceiptBegins} must be less than or equal to 75`);
  }
}

function geometricSeries(
  firstTerm: number,
  commonRatio: number,
  numberOfTerms: number,
) {
  if (commonRatio === 1) {
    return firstTerm * numberOfTerms;
  } else if (commonRatio > 1) {
    // Geometric series formula
    return firstTerm * (1 - commonRatio ** (numberOfTerms)) / (1 - commonRatio);
  } else {
    throw new RangeError(`commonRatio ${commonRatio} must be greater than or equal to 1.`);
  }
}

/**
 * @returns Rate due to delayed payment
 */
function rateByDelay(yearReceiptBegins: number): number {
  // Error check
  verifyYearReceiptBegins(yearReceiptBegins);

  // https://www.nenkin.go.jp/service/jukyu/roureinenkin/kuriage-kurisage/20140421-01.html
  const monthlyDecreaseRate = -0.004;

  // https://www.nenkin.go.jp/service/jukyu/roureinenkin/kuriage-kurisage/20140421-02.html
  const monthlyIncreaseRate= 0.007;

  const baseYear = 65;
  const yearDelay = yearReceiptBegins - baseYear;
  if (yearDelay < 0) {
    return 1 + Math.abs(yearDelay) * monthlyDecreaseRate * 12;
  } else {
    return 1 + yearDelay * monthlyIncreaseRate * 12;
  }
}


export default Vue.extend({
  name: 'Chart',
  data() {
    return {
      chartColorScheme: d3.schemeDark2,
      chartDatum: [] as chartData[],
      maxYearShown: 100,
      minYearShown: 1,
      maxYearLimit: MAX_YEAR_LIMIT,
      minYearLimit: MIN_YEAR_LIMIT,
      maxApy: MAX_APY,
      minApy: MIN_APY,
      maxMonthlyPayment: 200010,
      minMonthlyPayment: 17010, // contribution cost for national pension with additional pension
      apy: 0.01,
      monthlyPayment: 60010, // employees grade 32 minimum = 59475
      yearReceiptBegins: 65,
      nationalOptionNames: [
        {
          value: 'exemptFull',
          name: '全額免除',
        },
        {
          value: 'exempt34',
          name: '3/4免除',
        },
        {
          value: 'exempt12',
          name: '1/2免除',
        },
        {
          value: 'exempt14',
          name: '1/4免除',
        },
        {
          value: 'standard',
          name: '普通',
        },
        {
          value: 'additive',
          name: '付加年金',
        },

      ] as { value: nationalPensionOption, name: string}[],
      grades: d3.range(1, 33),
    }
  },
  watch: {
    apy(after, before) {
      this.applyToCharts({apy: after});
    },
    monthlyPayment(after, before) {
      this.applyToCharts({monthlyPayment: after});
    },
    yearReceiptBegins(after, before) {
      this.applyToCharts({yearReceiptBegins: after});
    },
  },
  methods: {
    applyToCharts(
      keyValue:
      { apy: number } |
      { monthlyPayment: number } |
      { yearReceiptBegins: number }
    ) {
      let key: 'apy' | 'monthlyPayment' | 'yearReceiptBegins';
      let value: number;
      if ('apy' in keyValue) {
        key = 'apy';
        value = keyValue['apy'];
      } else if ('monthlyPayment' in keyValue) {
        key = 'monthlyPayment';
        value = keyValue['monthlyPayment'];
      } else if ('yearReceiptBegins' in keyValue) {
        key = 'yearReceiptBegins';
        value = keyValue['yearReceiptBegins'];
      } else {
        exhaustiveCheck(keyValue, `all case not handled for ${keyValue}`);
      }

      for (const chartData of this.chartDatum) {
        if (key in chartData) {
          // @ts-ignore: key is key of chartData checked by `in` operator
          chartData[key] = value;
        }
        this.updateChartData(chartData);
      }
      this.renderChart();
    },
    renderChart() {
      // Error check
      if (!(this.$refs.chart instanceof SVGSVGElement)) {
        throw new Error('svg not found.');
      }
      const svg = d3.select(this.$refs.chart);
      // Delete previous chart
      svg.selectAll('*').remove();

      const minYear = this.minYearShown;
      const maxYear = this.maxYearShown;
      const years = d3.range(minYear, maxYear);
      const valuedChartDatum = this.chartDatum.filter((d) => d.data !== null) as valued<chartData>[];
      const datum = valuedChartDatum.map((d) => d.data.slice(minYear, maxYear));

      // Graph style settings
      const height = 500;
      const width = 720;
      const margin = {top: 40, right: 10, bottom: 45, left: 60};
      const innerWidth = width - margin.left - margin.right;
      const axisColor = '#2c2c2c';
      const axisFontSize = '10px';
      const labelColor = '#2c2c2c';
      const labelFontSize = '10px';

      // Create the graph
      svg.attr('viewBox', `0 0 ${width} ${height}`);

      const x = d3.scaleLinear()
        .domain([minYear, maxYear])
        .range([margin.left, width - margin.right]);
      const yMin = d3.min(datum.flat()) || 0;
      const yMax = d3.max(datum.flat()) || 0;
      const y = d3.scaleLinear()
        .domain([yMin, yMax])
        .range([height - margin.bottom, margin.top]);

      const xAxis = (g: d3.Selection<SVGGElement, unknown, any, any>) => {
        g.attr('transform', `translate(0,${height - margin.bottom})`)
          .call(d3.axisBottom(x).ticks(width / 80).tickSizeOuter(0));
        g.selectAll('line')
          .attr('stroke', axisColor)
          .attr('stroke-width', '0.3');
        g.selectAll('path')
          .attr('stroke', axisColor);
        g.selectAll('text')
          .attr('fill', axisColor)
          .attr('font-size', axisFontSize)
          .attr('preserveAspectRatio', 'xMinYMin');
      }
      const yAxis = (g: d3.Selection<SVGGElement, unknown, any, any>) => {
        g.attr('transform', `translate(${margin.left},0)`)
          .call(
            d3.axisLeft(y)
              .tickFormat((d) => Number(d) > 10000 ? String(Number(d) / 10000) : String(d))
          )
          .call((g: d3.Selection<SVGGElement, unknown, any, any>) => g.select('.domain').remove());
        g.selectAll('line')
          .attr('stroke', axisColor);
        g.selectAll('text')
          .attr('fill', axisColor)
          .attr('font-size', axisFontSize);
      }
      const yAxisGrid = (g: d3.Selection<SVGGElement, unknown, any, any>) => {
        g.attr('transform', `translate(${margin.left},0)`)
          .call(
            d3.axisLeft(y)
              .ticks(height / 40)
              .tickSize(-innerWidth)
              .tickFormat(() => '')
          )
          .call((g: d3.Selection<SVGGElement, unknown, any, any>) => g.select('.domain').remove())
          .selectAll('line')
          .attr('stroke-opacity', '1')
          .attr('stroke-width', '0.1')
          .attr('stroke', axisColor);
      }
      svg.insert('g').call(xAxis);
      svg.insert('g').call(yAxis);
      svg.insert('g').call(yAxisGrid);

      // x axis label
      svg.append('text')
        .text('(20歳から経過した年)')
        .attr('text-anchor', 'end')
        .attr('x', width - margin.right)
        .attr('y', height - margin.bottom + 35)
        .attr('fill', labelColor)
        .attr('font-size', labelFontSize)
        .attr('font-family', 'sans-serif')
        .attr('font-weight', 'normal');

      // y axis label
      svg.append('text')
        .text('(万円)')
        .attr('text-anchor', 'end')
        .attr('x', margin.left)
        .attr('y', margin.top - 10)
        .attr('fill', labelColor)
        .attr('font-size', labelFontSize)
        .attr('font-family', 'sans-serif')
        .attr('font-weight', 'normal');

      // Get chartDatum for each chart type
      const noPensionDatum: valued<noPensionData>[] = [];
      const nationalPensionDatum: valued<nationalPensionData>[] = [];
      const employeesPensionDatum: valued<employeesPensionData>[] = [];
      for (const chartData of valuedChartDatum) {
        if (chartData.data === null) continue;
        switch (chartData.type) {
          case 'noPension':
            noPensionDatum.push(chartData as valued<noPensionData>);
            break;
          case 'nationalPension':
            nationalPensionDatum.push(chartData as valued<nationalPensionData>);
            break;
          case 'employeesPension':
            employeesPensionDatum.push(chartData as valued<employeesPensionData>);
            break;
        }
      }

      // Plot function declaration
      type dasharray = number[] | 'none';
      const line = d3.line()
        .x((v) => x(v[0]))
        .y((v) => y(v[1]));
      function plotPath({
        chartData,
        width,
        dasharray,
      }: {
        chartData: valued<chartData>,
        width: number,
        dasharray: dasharray,
      }) {
        chartData
        svg.append('path')
          .attr('d', line(years.map((v, i) => [v, chartData.data[i]])))
          .attr('fill', 'none')
          .attr('stroke', chartData.color)
          .attr('stroke-opacity', '1')
          .attr('stroke-width', width)
          .attr('stroke-dasharray', dasharray)
          .attr('stroke-miterlimit', 1) // mitigate spikes
          .attr('stroke-linecap', 'round');
      }

      // Plot chart datum

      // The front chartDatum comes the front of the chart
      const orderedChartDatum = [nationalPensionDatum, employeesPensionDatum, noPensionDatum];
      for (const chartDatum of orderedChartDatum.reverse()) {
        if (chartDatum.length === 0) continue;
        let width: number;
        let dasharray: dasharray;
        switch (chartDatum[0].type) {
          case 'noPension':
            width = 1.2;
            dasharray = [6, 8];
            break;
          case 'nationalPension':
            width = 1.5;
            dasharray = [1, 4];
            break;
            case 'employeesPension':
            width = 0.8;
            dasharray = 'none';
            break;
        }
        for (const chartData of chartDatum) {
          plotPath({ chartData, width, dasharray });
        }
      }


    },
    updateChartColors() {
      // Get datum for each pension type
      const noPensionDatum = this.chartDatum.filter((d) => d.type === 'noPension') as noPensionData[];
      const nationalPensionDatum = this.chartDatum.filter((d) => d.type === 'nationalPension') as nationalPensionData[];
      const employeesPensionDatum = this.chartDatum.filter((d) => d.type === 'employeesPension') as employeesPensionData[];

      // no-pension datum
      for (const chartData of noPensionDatum) {
        chartData.color = '#000';
      }

      // national-pension datum
      const nationalCmap = subCmap(d3.interpolateYlOrRd, 0.2, 0.8, true);
      for (const [i, chartData] of nationalPensionDatum.entries()) {
        const color = nationalCmap(i / (nationalPensionDatum.length - 1));
        chartData.color = color;
      }

      // employees-pension datum
      const employeesCmap = subCmap(d3.interpolatePuBu, 0.25, 0.95, true);
      for (const [i, chartData] of employeesPensionDatum.entries()) {
        const color = employeesCmap(i / (employeesPensionDatum.length - 1));
        chartData.color = color;
      }
    },
    updateChartData(chartData: chartData) {
      let newData: number[] | null;
      try {
        switch (chartData.type) {
          case 'noPension':
            newData = noPension({...chartData} as noPensionData);
            break;
          case 'nationalPension':
            newData = nationalPension({...chartData} as nationalPensionData);
            break;
          case 'employeesPension':
            newData = employeesPension({...chartData} as employeesPensionData);
            break;
          default:
            exhaustiveCheck(chartData, `Unexpected pension type of chartData ${chartData}`);
        }
      } catch (err) {
        if (err instanceof Error) {
          chartData.error = err;
        } else {
          console.error(`Unexpected err ${err} in updateChartData`);
        }
        newData = null;
      }
      chartData.data = newData;
    },
  },
  mounted () {
      const apy = this.apy;
      const monthlyPayment = this.monthlyPayment;
      const yearInvestmentEnds = this.maxYearLimit;
      const yearReceiptBegins = this.yearReceiptBegins;

      // Add no-pension data
      const noPensionData: noPensionData = {
        name: '年金なし',
        type: 'noPension',
        apy,
        monthlyPayment,
        yearInvestmentEnds,
        color: '',
        data: null,
        error: null,
      }
      this.chartDatum.push(noPensionData);

      // Add national-pension data
      for (const {value: optionValue, name: optionName} of this.nationalOptionNames) {
        const nationalPensionData: nationalPensionData = {
          name: `国民年金(${optionName})`,
          type: 'nationalPension',
          apy,
          monthlyPayment,
          yearInvestmentEnds,
          color: '',
          data: null,
          error: null,
          yearReceiptBegins,
          option: optionValue,
        };
        this.chartDatum.push(nationalPensionData);
      }

      // Add national-pension data
      for (const grade of this.grades) {
        const employeesPensionData: employeesPensionData = {
          name: `厚生年金(等級${grade})`,
          type: 'employeesPension',
          apy,
          monthlyPayment,
          yearInvestmentEnds,
          color: '',
          data: null,
          error: null,
          yearReceiptBegins,
          grade,
          includeNational: true,
        };
        this.chartDatum.push(employeesPensionData);
      }

      for (const chartData of this.chartDatum) {
        this.updateChartData(chartData);
      }
      this.updateChartColors();
      this.renderChart();
  },
})
</script>