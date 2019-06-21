<template>
  <div class="clamc-oms-ui fixedIncome standardTable">
    <suspendWin v-if="markFlag" :title='markTitle' :isShow='isShow' @colseMark = 'colseMark'></suspendWin>
    <!-- 时间 -->
    <section class="searchbox">
      <div class="right">
        <el-date-picker v-model="sdate"
          value-format="yyyy-MM-dd"
          format="yyyy 年 MM 月 dd 日"
          type="date"
          :picker-options="sdateOptions"
          placeholder="选择日期"
          @change="sdateChange"></el-date-picker>
      </div>
    </section>
    <section class="body">
      <!-- 主要指标 -->
      <div class="indices"
        v-loading="indicesLoading">
        <el-row :gutter="11">
          <el-col :md="6">
            <panel noTitle>
              <card :data="indices[0]"
                titleMark="受托系统内"
                changeUnit="亿"
                text="较年初变动" />
            </panel>
          </el-col>
          <el-col :md="6">
            <panel noTitle
              class="indicesCard">
              <IndicesCard :data="indices[1]" />
              <IndicesCard :data="indices[2]"
                style="margin-top: 10px"
                isActual />
            </panel>
          </el-col>
          <el-col :md="6">
            <panel noTitle
              class="indicesCard">
              <IndicesCard :data="indices[3]"
                unit="%" />
              <IndicesCard :data="indices[4]"
                style="margin-top: 10px"
                unit="%"
                isActual />
            </panel>
          </el-col>
          <el-col :md="6">
            <panel noTitle
              class="indicesCard">
              <IndicesCard :data="indices[5]" />
              <IndicesCard :data="indices[6]"
                style="margin-top: 10px"
                changeUnit="亿"
                text="同比变动"
                isActual />
            </panel>
          </el-col>
        </el-row>
      </div>
      <!-- 总览、分账户 -->
      <div class="profit polymerization-panel">
        <panel noTitle>
          <el-tabs v-model="profitActive">
            <el-tab-pane label="总览"
              name="overview">
              <div class="tab-bar table-on-search">
                <el-form :inline="true"
                    ref="overviewForm"
                    :model="overviewForm">
                    <el-form-item prop="date">
                    <el-date-picker v-model="overviewForm.date"
                        value-format="yyyy-MM-dd"
                        format="yyyy 年 MM 月 dd 日"
                        placeholder="请选择日期"/>
                    </el-form-item>
                </el-form>
                <div class="tab-right">
                    <icon-tip content="下载" name="icon_download" class="moreicon" @click="overviewExport()"></icon-tip>
                </div>
              </div>
              
              <TreeTable :columns="overviewColumns"
                :loading="overviewLoading"
                :tree-structure="true"
                :defaultExpandAll="false"
                :data-source="overviewData" />
            </el-tab-pane>
            <el-tab-pane label="分账户"
              name="account">
              <div class="tab-bar table-on-search custom-box-message">
                  <el-form :inline="true"
                    style="padding-left:10px"
                    v-model="accountForm">
                    <el-form-item label="指标:">
                    <el-select v-model="accountForm.index">
                        <el-option v-for="(item, index) in indexList"
                        :key="index"
                        :value="item"></el-option>
                    </el-select>
                    </el-form-item>
                    <el-form-item>
                    <el-button
                        class="defaultBtn"
                        @click="$refs['accountDialog'].showDialog()">选择账户</el-button>
                    <Dialog ref="accountDialog"
                        title="账户选择"
                        dataType="account"
                        :node="true"
                        :data="accountTreeLevelFourNewData"
                        :checkStrictly='true'
                        :showInfo=showInfoFlag
                        @onOk="accountAccountOnok($event, 'accountForm')" />
                    </el-form-item>
                    <el-form-item>
                    <el-button
                        type="primary"
                        class="defaultBtn"
                        @click="accountSearch(accountForm)">确定</el-button>
                    </el-form-item>
                </el-form>
                <div class="tab-right">
                    <icon-tip content="下载" name="icon_download" class="moreicon" @click="accountExport(accountForm)"></icon-tip>
                </div>
              </div>
              <tree-table :columns="accountColumns"
                :loading="accountLoading"
                :defaultExpandAll="false"
                :tree-structure="true"
                :data-source="accountData" />
            </el-tab-pane>
          </el-tabs>
        </panel>
      </div>
      <!-- 新增配置 -->
      <div class="newConfiguration polymerization-panel custom-box-message table-on-search">
        <panel title="新增配置">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('newConfiguration')"></icon-tip>
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('newConfiguration')"></icon-tip>
          </template>
          <el-form :inline="true"
            ref="newConfigForm"
            :model="newConfigForm">
            <el-form-item prop="date"
              :rules="[{required: true, message: '请选择日期', trigger: 'blur'}]">
              <el-date-picker v-model="newConfigForm.date"
                type="daterange"
                range-separator="至"
                value-format="yyyy-MM-dd"
                start-placeholder="开始日期"
                end-placeholder="结束日期"></el-date-picker>
            </el-form-item>
            <el-form-item label="会计分类:"
              prop="categoriesOfAccounting">
              <el-select multiple collapse-tags v-model="newConfigForm.categoriesOfAccounting">
                <el-option v-for="(item, index) in categoriesOfAccounting"
                  :key="index"
                  :label="item.combName"
                  :value="item.combCode"/>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button
                class="defaultBtn"
                @click="$refs['newConfigDialog'].showDialog()">选择账户</el-button>
              <Dialog ref="newConfigDialog"
                title="账户选择"
                dataType="account"
                :data="accountTreeLevelFourData"
                @onOk="accountOnOk($event ,'newConfigForm')" />
            </el-form-item>
            <el-form-item>
              <el-button
                type="primary"
                class="defaultBtn"
                :loading="newConfigLoading"
                @click="newConfigSearch(newConfigForm, 'newConfigForm')">{{newConfigLoading?'加载中':'确定'}}</el-button>
            </el-form-item>
          </el-form>
          <tree-table :columns="newConfigColumns"
            :tree-structure="true"
            span="系统内合计"
            :loading="newConfigLoading"
            :data-source="newConfigData" />
        </panel>
      </div>
      <!-- 持仓情况 -->
      <div class="position">
        <panel title="持仓情况" class="polymerization-panel table-on-search custom-box-message">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('position')"></icon-tip>
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('position')"></icon-tip>
          </template>
          <Dialog ref="positionAssetType"
            title="资产类别选择"
            dataType="assetsType"
            :data="assetsType"
            @onOk="assetsTypeOnOk($event, 'positionForm')" />
          <Dialog ref="positionDialog"
            title="账户选择"
            dataType="account"
            :data="accountTreeLevelFourNewData"
            :checkStrictly='true'
            :showInfo=showInfoFlag
            @onOk="accountOnOk($event, 'positionForm')" />
          <el-tabs class="positionTabs"
            v-model="positionActiveName"
            type="border-card"
            style="margin-bottom:0px">
            <el-tab-pane label="按账户"
              :name="1">
              <el-form :model="positionForm"
                :inline="true">
                <el-form-item>
                  <el-date-picker placeholder="请选择日期"
                    :clearable="false"
                    value-format="yyyy-MM-dd"
                    format="yyyy 年 MM 月 dd 日"
                    v-model="positionForm.date" />
                </el-form-item>
                <el-form-item>
                  <el-button 
                    class="defaultBtn"
                    @click="$refs['positionAssetType'].showDialog()">资产类别</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    class="defaultBtn"
                    @click="$refs['positionDialog'].showDialog()">选择账户</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    type="primary"
                    class="defaultBtn"
                    :loading="positionLoading"
                    @click="positionSearch(positionForm)">{{positionLoading?'加载中':'确定'}}</el-button>
                </el-form-item>
              </el-form>
              <tree-table :columns="positionByAccountColumns"
                :tableProps="{ 'show-summary': false }"
                :tree-structure="true"
                :defaultExpandAll="false"
                :loading="positionLoading"
                :data-source="positionData" />
            </el-tab-pane>
            <el-tab-pane label="按会计分类"
              :name="2">
              <el-form :model="positionForm"
                :inline="true">
                <el-form-item>
                  <el-date-picker placeholder="请选择日期"
                    :clearable="false"
                    value-format="yyyy-MM-dd"
                    format="yyyy 年 MM 月 dd 日"
                    v-model="positionForm.date" />
                </el-form-item>
                <el-form-item>
                  <el-button 
                    class="defaultBtn"
                    @click="$refs['positionAssetType'].showDialog()">资产类别</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    class="defaultBtn"
                    @click="$refs['positionDialog'].showDialog()">选择账户</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    type="primary"
                    class="defaultBtn"
                    :loading="positionLoading"
                    @click="positionSearch(positionForm)">{{positionLoading?'加载中':'确定'}}</el-button>
                </el-form-item>
              </el-form>
              <Table :dataSource="positionData"
                :columns="positionByAccountingColumns"
                :height="false"
                :loading="positionLoading" />
            </el-tab-pane>
            <el-tab-pane label="按账户&按会计分类"
              :name="3">
              <el-form :model="positionForm"
                :inline="true">
                <el-form-item>
                  <el-date-picker placeholder="请选择日期"
                    :clearable="false"
                    value-format="yyyy-MM-dd"
                    format="yyyy 年 MM 月 dd 日"
                    v-model="positionForm.date" />
                </el-form-item>
                <el-form-item>
                 <el-button 
                    class="defaultBtn"
                    @click="$refs['positionAssetType'].showDialog()">资产类别</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    class="defaultBtn"
                    @click="$refs['positionDialog'].showDialog()">选择账户</el-button>
                </el-form-item>
                <el-form-item>
                  <el-button
                    type="primary"
                    class="defaultBtn"
                    :loading="positionLoading"
                    @click="positionSearch(positionForm)">{{positionLoading?'加载中':'确定'}}</el-button>
                </el-form-item>
              </el-form>
              <Table :dataSource="positionData"
                :config="{ 'span-method': objectSpanMethod, 'max-height': 330 }"
                :columns="positionByAllColumns"
                :height="false"
                :loading="positionLoading"
                class="borderSetting"/>
            </el-tab-pane>
          </el-tabs>
        </panel>
      </div>
      <!-- 交易情况 -->
      <div class="transaction">
        <panel title="交易情况" class="polymerization-panel table-on-search custom-box-message">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('transaction')"></icon-tip>
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('transaction')"></icon-tip>
          </template>
          <el-form :model="transactionForm"
            ref="transactionForm"
            :inline="true">
            <el-form-item prop="date"
              :rules="[{required: true, message: '请选择日期', trigger: 'blur'}]">
              <el-date-picker v-model="transactionForm.date"
                type="daterange"
                range-separator="至"
                value-format="yyyy-MM-dd"
                start-placeholder="开始日期"
                end-placeholder="结束日期"></el-date-picker>
            </el-form-item>
            <el-form-item label="会计分类:"
              prop="categoriesOfAccounting">
              <el-select multiple collapse-tags v-model="transactionForm.categoriesOfAccounting">
                <el-option v-for="(item, index) in categoriesOfAccounting"
                  :key="index"
                  :label="item.combName"
                  :value="item.combCode"/>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button 
                    class="defaultBtn"
                    @click="$refs['transactionAssetType'].showDialog()">资产类别</el-button>
              <Dialog ref="transactionAssetType"
                    title="资产类别选择"
                    dataType="account"
                    :data="assetsType"
                    @onOk="assetsTypeOnOk($event, 'transactionForm')" />
            </el-form-item>
            <el-form-item>
              <el-button
                class="defaultBtn"
                @click="$refs['transactionDialog'].showDialog()">选择账户</el-button>
              <Dialog ref="transactionDialog"
                title="账户选择"
                dataType="account"
                :data="accountTreeLevelFourData"
                @onOk="accountOnOk($event ,'transactionForm')" />
            </el-form-item>
            <el-form-item>
              <el-button
                type="primary"
                class="defaultBtn"
                :loading="transactionLoading"
                @click="transactionSearch(transactionForm, 'transactionForm')">{{transactionLoading?'加载中':'确定'}}</el-button>
            </el-form-item>
          </el-form>
          <div class="standardTable">
              <el-table :data="transactionData"
                v-loading="transactionLoading">
                <el-table-column prop="assetType"
                label="交易类别" />
                <el-table-column align="right"
                prop="amount"
                label="成交金额(亿)"
                :formatter="thousandthFormatter" />
                <el-table-column align="right"
                prop="averageIncome"
                label="加权平均收益率"
                :formatter="percentFormatter" />
                <el-table-column align="right"
                prop="remainderPeriod"
                label="加权平均剩余期限"
                :formatter="thousandthFormatter" />
                <el-table-column align="right"
                prop="profitAndLoss"
                label="实现盈亏(亿)"
                :formatter="thousandthFormatter" />
            </el-table>
          </div>
        </panel>
      </div>
      <!-- 存量资产预计到期情况 -->
      <div class="expectedOfStockAssets">
        <panel title="存量资产预计到期情况" class="polymerization-panel  table-on-search custom-box-message">
          <template slot="title-right">
            <!-- <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('expectedAssets')"></icon> -->
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('expectedOfStockAssets')"></icon-tip>
          </template>
          <bar-of-expected-assets ref="barOfExpectedAssets" :sdate="sdate" />
        </panel>
      </div>
      <!-- 品种债剩余期限 -->
      <div class="varietyDebtRemainingPeriod">
        <panel title="品种债剩余期限" class="polymerization-panel custom-box-message">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('varietyDebt')"></icon-tip>
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('varietyDebtRemainingPeriod')"></icon-tip>
          </template>
          <div class="standardTable">
              <el-table :data="varietyDebtData"
                :cell-style="cellStyle"
                :max-height="360"
                v-loading="varietyDebtLoading"
                :span-method="spanMethod">
                <el-table-column label="品种债"
                show-overflow-tooltip
                class="spancolumn"
                prop="varietyDebt"></el-table-column>
                <el-table-column label="指标"
                show-overflow-tooltip
                prop="index"
                width="150" />
                <el-table-column label="1年内"
                show-overflow-tooltip
                :formatter="debtFormatter"
                align="right"
                prop="year1"></el-table-column>
                <el-table-column label="1-3年"
                show-overflow-tooltip
                :formatter="debtFormatter"
                align="right"
                prop="year1to3" />
                <el-table-column label="3-5年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year3to5" />
                <el-table-column label="5-7年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year5to7" />
                <el-table-column label="7-10年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year7to10" />
                <el-table-column label="10-20年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year10to20" />
                <el-table-column label="20-30年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year20to30" />
                <el-table-column label="30-50年"
                :formatter="debtFormatter"
                show-overflow-tooltip
                align="right"
                prop="year30to50" />
            </el-table>
          </div>
        </panel>
      </div>
      <!-- 交易盘 -->
      <div class="transactionPan">
        <panel title="交易盘" class="polymerization-panel custom-box-message">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon" 
              @click="download('transactionpan')"/>
          </template>
          <div class="standardTable">
              <el-table v-loading="transactionPanLoading"
                :data="transactionPanData">
                <el-table-column prop="name"
                label="组合名称"
                show-overflow-tooltip />
                <el-table-column align="right"
                prop="endScale"
                label="期末规模(亿)"
                :formatter="thousandthFormatter" />
                <el-table-column align="right"
                prop="profit"
                :formatter="thousandthFormatter"
                label="实现收益(减值后)(亿)" />
                <el-table-column align="right"
                prop="yield"
                :formatter="percentFormatter"
                label="实现收益率(减值后)" />
                <!-- <el-table-column align="right"
                prop="netWorth"
                label="期末单位净值"
                :formatter="thousandthFormatter" /> -->
            </el-table>
          </div>
        </panel>
      </div>
      <!-- 管理第三方产品情况 -->
      <div class="thirdPartyProducts">
        <panel title="管理第三方产品情况" class="polymerization-panel table-on-search">
          <template slot="title-right">
            <icon-tip content="下载" name="icon_download"
              class="moreicon"
              @click="download('thirdPartyProducts')"></icon-tip>
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('thirdPartyProducts')"></icon-tip>
              <!-- @click="moreClick('thirdPartyProducts', '管理第三方产品情况')" -->
          </template>
            <div style="padding-bottom: 10px;">
                <el-date-picker
                    v-model="insureDate"
                    value-format="yyyy-MM-dd"
                    format="yyyy 年 MM 月 dd 日"
                    type="date"
                    placeholder="选择日期"
                    @change="insureDateChange"
                ></el-date-picker>
            </div>
          <Table :dataSource="thirdPartyProductsData"
            :height="false"
            :columns="thirdPartyProductsColumns"
            :loading="thirdPartyProductsLoading" />
        </panel>
      </div>
      <!-- 委托资金变化明细 -->
      <div class="changeOfFund">
        <panel title="委托资金余额变化" class="polymerization-panel custom-box-message">
          <template slot="title-right">
            <icon-tip content="更多" name="icon_more"
              class="moreicon"
              @click="moreClick('changeOfFund')" />
          </template>
          <bar-of-fund-change :sdate="sdate" />
        </panel>
      </div>
    </section>
  </div>
</template>

<script>
import Vue from 'vue'
import moment from 'moment';
import debounce from 'lodash/debounce';
import Panel from '@components/Panel';
import Card from '../card';
import Table from '@components/Table';
import IndicesCard from './indicesCard';
import TreeTable from '../TreeTable';
// import TreeTable from '@components/TreeTable';
import barOfFundChange from './barOfFundChange';
import barOfExpectedAssets from './barOfExpectedAssets';
import Dialog from '../dialog';
import { currencyFormatter } from '@helper/commonUtil';
import {
  getClassification,
  getAssetType,
  getAccountTree,
  getAccountTreeNew,
  getAccountTreeN4 as getAccountTreeLevelFour
 } from '../../../api/navigate/common';
import { getDate, setDate } from '../storageDate';
import { getIndices,
  getAccount,
  getVarietyDebt,
  getTransactionpan,
  getThirdPartyProducts,
  getTransaction,
  getProfitOverview,
//   getProfitAccount,
  getProfitAccountNew,
  exportProfitOverview,
  exportProfitAccount,
  getPosition,
  getNewConfiguration,
  exportNewConfiguration,
  exportPosition,
  exportTransaction,
  exportExpectedAssets,
  exportVarietyDebt,
  exportTransactionpan,
  exportThirdPartyProducts
  } from '../../../api/navigate/fixedIncome';
import { getWorkDay } from '../../../api/navigate/financeMarkets';
import { getTips } from '../../../api/navigate/navigate';
// import { selectorAll } from 'd3-selection';
const currency = function(row, column, cellValue, index) {
    if (typeof cellValue !== 'number') {
        return cellValue;
    }
    return currencyFormatter(cellValue);
};

const currency44 = function(row, column, cellValue, index) {
    return currencyFormatter(cellValue, '', 4);
};

const accountType = {
    DIVIDEACCOUNT: 'divideAccount',
    NEWCOFIGURATION: 'newConfiguration',
    POSITION: 'position',
    TRANSACTION: 'transaction'
};
const indexList = [
    '期末规模', '规模占比',
    '目标实现收益', '实现收益率(减值后)',
    '实现收益(减值后)', '综合收益',
    '综合收益率'
];
const overviewColumns = [
    {
        label: '',
        prop: 'deptName',
        'min-width': 200
    }, {
        label: '期末规模(亿)',
        prop: 'endSize',
        align: 'right',
        render: currencyFormatter
    }, {
        label: '规模占比',
        prop: 'scaleProportion',
        align: 'right',
        render: percentFormatter
    }, {
        label: '目标实现收益(亿)',
        prop: 'profit',
        align: 'right',
        'min-width': 120,
        render: currencyFormatter
    }, {
        label: '实现收益率(减值后)',
        prop: 'yieldRate',
        align: 'right',
        'min-width': 130,
        render: percentFormatter
    }, {
        label: '实现收益(减值后)(亿)',
        prop: 'achieveProfit',
        'min-width': 120,
        align: 'right',
        render: currencyFormatter
    }, {
        label: '综合收益(亿)',
        prop: 'fullIncome',
        align: 'right',
        render: currencyFormatter
    }, {
        label: '综合收益率',
        prop: 'fullIncomeRate',
        align: 'right',
        'min-width': 120,
        render: percentFormatter
    }
];
const newConfigColumns = [
    {
        label: '资产类型',
        prop: 'assetType',
        'min-width': 130,
        'header-align': 'center',
        fixed: true
    },
    {
        label: '系统内合计',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate',
                'min-width': 110,
                align: 'right'
            }
        ]
    },
    {
        label: '集团',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale_jt',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield_jt',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield_jt',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield_jt',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield_jt',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod_jt',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit_jt',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit_jt',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate_jt',
                'min-width': 110,
                align: 'right'
            }
        ]
    },
    {
        label: '万能',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale_wn',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield_wn',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield_wn',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield_wn',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield_wn',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod_wn',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit_wn',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit_wn',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate_wn',
                'min-width': 110,
                align: 'right'
            }
        ]
    },
    {
        label: '传统',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale_ct',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield_ct',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield_ct',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield_ct',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield_ct',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod_ct',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit_ct',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit_ct',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate_ct',
                'min-width': 110,
                align: 'right'
            }
        ]
    },
    {
        label: '分红',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale_fh',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield_fh',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield_fh',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield_fh',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield_fh',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod_fh',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit_fh',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit_fh',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate_fh',
                'min-width': 110,
                align: 'right'
            }
        ]
    },
    {
        label: '财险',
        'header-align': 'center',
        column: [
            {
                label: '新增配置规模(亿)',
                prop: 'configureScale_cx',
                'min-width': 140,
                align: 'right',
                render: currencyFormatter
            }, {
                label: '目标新增配置平均收益率',
                prop: 'targetYield_cx',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '实际新增配置平均收益率',
                prop: 'realYield_cx',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税后收益率',
                prop: 'VATYield_cx',
                'min-width': 150,
                align: 'right',
                render: percentFormatter
            }, {
                label: '增值税、所得税后收益率',
                prop: 'VATIncomeYield_cx',
                'min-width': 120,
                align: 'right',
                render: percentFormatter
            }, {
                label: '新增配置平均剩余期限',
                prop: 'remainderPeriod_cx',
                'min-width': 100,
                align: 'right'
            }, {
                label: '配置额度(亿)',
                prop: 'allocationLimit_cx',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度(亿)',
                prop: 'remainderLimit_cx',
                align: 'right',
                'min-width': 100,
                render: currencyFormatter
            }, {
                label: '剩余额度占比',
                prop: 'remainderRate_cx',
                'min-width': 110,
                align: 'right'
            }
        ]
    }
];
const thirdPartyProductsColumns = [
    {
        prop: 'productName',
        label: '产品名称',
        'min-width': 130,
        'show-overflow-tooltip': true
    }, {
        prop: 'startTime',
        label: '起始运作时间',
        'min-width': 150,
        align: 'right',
        'show-overflow-tooltip': true
    }, {
        prop: 'type',
        label: '类型',
        'min-width': 160,
        'show-overflow-tooltip': true
    },
    // {
    //     prop: 'accountingMethod',
    //     label: '会计核算方法',
    //     'min-width': 150,
    //     'show-overflow-tooltip': true
    // },
    {
        prop: 'endNetValue',
        label: '期末净值(亿)',
        'min-width': 130,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    }, {
        prop: 'share',
        label: '份额(万份)',
        'min-width': 150,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    }, {
        prop: 'unitNetValue',
        label: '单位净值',
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency44
    }, {
        prop: 'client',
        label: '委托人',
        'min-width': 150,
        'show-overflow-tooltip': true
    }, {
        prop: 'custodian',
        label: '托管人',
        'min-width': 150,
        'show-overflow-tooltip': true
    }, {
        prop: 'capitalInflow',
        label: '本年资金流入(亿)',
        'min-width': 130,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    },
    {
        prop: 'netValueRaise',
        label: '本年以来净值增长率',
        'min-width': 150,
        align: 'right',
        formatter: percentFormatter,
        'show-overflow-tooltip': true
    },
    {
        prop: 'benchmark',
        label: '业绩基准',
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    }, {
        prop: 'managementFee',
        label: '产品管理费(元)',
        'min-width': 130,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    }, {
        prop: 'sumNetValue',
        label: '累计单位净值',
        'min-width': 110,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency44
    }, {
        prop: 'repoAmount',
        label: '正回购余额(亿)',
        'min-width': 130,
        align: 'right',
        'show-overflow-tooltip': true,
        formatter: currency
    }
];
const fixed2 = function (num, l = 2) {
    return Number.prototype.toFixed.call(num, 2);
};
// 树形表格用
function percentFormatter(cellValue) {
    if (typeof cellValue !== 'number') {
        return cellValue;
    }
    return fixed2(cellValue) + '%';
}
// 普通表格
function percent(row, column, cellValue, index) {
    if (typeof cellValue !== 'number') {
        return cellValue;
    }
    return fixed2(cellValue) + '%';
}
function thousandthFormatter(row, column, cellValue, index) {
    return currencyFormatter(cellValue);
};
// 页面初始日期
const beginDate = moment().subtract(1, 'days').format('YYYY-MM-DD');
const newDate = moment().subtract(2, 'days').format('YYYY-MM-DD');
export default {
    components: {
        panel: Panel,
        Card,
        Table,
        IndicesCard,
        TreeTable,
        barOfFundChange,
        barOfExpectedAssets,
        Dialog
    },
    data() {
        return {
            sdate: beginDate,
            sdateOptions: {
                disabledDate(date) {
                    return Date.now() <= date;
                }
            },
            indices: [],
            accountType,
            activeAccountType: '',
            profitActive: 'overview',
            accountVisible: false,
            accountActiveName: '',   // 账户选择，折叠面板张开页
            positionActiveName: 1,
            accountDialogLoading: false,
            accountSelected: [],
            onSureAccount: {
                [accountType.DIVIDEACCOUNT]: [],
                [accountType.NEWCOFIGURATION]: [],
                [accountType.POSITION]: [],
                [accountType.TRANSACTION]: []
            },
            overviewForm: {
                keyword: '',
                date: ""
            },
            accountForm: {
                keyword: '',
                index: '',
                account: '',
                accountNames: ''
            },
            newConfigForm: {
                keyword: '',
                categoriesOfAccounting: [],
                date: ["", ""],
                account: ''
            },
            positionForm: {
                keyword: '',
                date: "",
                productCategory: '',
                account: ''
            },
            transactionForm: {
                keyword: '',
                date: ["", ""],
                categoriesOfAccounting: [],
                productCategory: '',
                account: ''
            },
            overviewColumns,
            accountColumns: [
                {
                    label: '',
                    prop: 'deptName',
                    'min-width': 200
                }
            ],
            newConfigColumns,
            positionByAccountColumns: [
                {
                    prop: 'account',
                    label: '账户',
                    width: 70,
                    align: 'right',
                    'show-overflow-tooltip': true
                }, {
                    label: '期末规模(亿)',
                    prop: 'endScale',
                    align: 'right',
                    render: currencyFormatter
                }, {
                    label: '实现收益率(减值后)',
                    prop: 'achieveYield',
                    align: 'right',
                    'min-width': 120,
                    render: percentFormatter
                }, {
                    label: '实现收益(减值后)(亿)',
                    prop: 'achieveProfit',
                    align: 'right',
                    render: currencyFormatter,
                    'min-width': 120
                }, {
                    label: '综合收益(亿)',
                    prop: 'fullProfit',
                    align: 'right',
                    render: currencyFormatter
                }, {
                    label: '综合收益率',
                    prop: 'fullYield',
                    align: 'right',
                    render: percentFormatter
                }, {
                    label: '剩余期限',
                    prop: 'remainderPeriod',
                    align: 'right'
                }
            ],
            positionByAccountingColumns: [
                {
                    prop: 'subject',
                    label: '科目'
                },
                {
                    label: '期末规模(亿)',
                    prop: 'endScale',
                    align: 'right',
                    formatter: thousandthFormatter
                }, {
                    label: '实现收益率(减值后)',
                    prop: 'achieveYield',
                    align: 'right',
                    'min-width': 120,
                    formatter: percent
                }, {
                    label: '实现收益(减值后)(亿)',
                    prop: 'achieveProfit',
                    align: 'right',
                    formatter: thousandthFormatter,
                    'min-width': 120
                }, {
                    label: '综合收益(亿)',
                    prop: 'fullProfit',
                    align: 'right',
                    formatter: thousandthFormatter
                }, {
                    label: '综合收益率',
                    prop: 'fullYield',
                    align: 'right',
                    formatter: percent
                }, {
                    label: '剩余期限',
                    prop: 'remainderPeriod',
                    align: 'right'
                }
            ],
            positionByAllColumns: [
                {
                    prop: 'account',
                    label: '账户',
                    'show-overflow-tooltip': true
                },
                {
                    prop: 'subject',
                    label: '科目',
                    'show-overflow-tooltip': true
                },
                {
                    label: '期末规模(亿)',
                    prop: 'endScale',
                    align: 'right',
                    formatter: thousandthFormatter
                }, {
                    label: '实现收益率(减值后)',
                    prop: 'achieveYield',
                    align: 'right',
                    'min-width': 120,
                    formatter: percent
                }, {
                    label: '实现收益(减值后)(亿)',
                    prop: 'achieveProfit',
                    align: 'right',
                    formatter: thousandthFormatter,
                    'min-width': 120
                }, {
                    label: '综合收益(亿)',
                    prop: 'fullProfit',
                    align: 'right',
                    formatter: thousandthFormatter
                }, {
                    label: '综合收益率',
                    prop: 'fullYield',
                    align: 'right',
                    formatter: percent
                }, {
                    label: '剩余期限',
                    prop: 'remainderPeriod',
                    align: 'right'
                }
            ],
            varietyDebtColumns: [ // 未使用
                {
                    prop: 'varietyDebt',
                    label: '品种债',
                    'show-overflow-tooltip': true,
                    'class-name': this.spancolumn
                }, {
                    prop: 'index',
                    label: '指标',
                    'show-overflow-tooltip': true,
                    'min-width': 150
                }, {
                    prop: 'year1',
                    label: '1年内',
                    'show-overflow-tooltip': true,
                    align: "right",
                    formatter: this.debtFormatter
                }, {
                    prop: 'year1to3',
                    label: '1-3年',
                    'show-overflow-tooltip': true,
                    formatter: this.debtFormatter,
                    align: 'right'
                }, {
                    prop: 'year3to5',
                    label: '3-5年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }, {
                    prop: 'year5to7',
                    label: '5-7年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }, {
                    prop: 'year7to10',
                    label: '7-10年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }, {
                    prop: 'year10to20',
                    label: '10-20年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }, {
                    prop: 'year20to30',
                    label: '20-30年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }, {
                    prop: 'year30to50',
                    label: '30-50年',
                    'show-overflow-tooltip': true,
                    align: 'right',
                    formatter: this.debtFormatter
                }
            ],
            thirdPartyProductsColumns,
            indicesLoading: false,
            overviewLoading: false,
            accountLoading: false,
            newConfigLoading: false,
            positionLoading: false,
            transactionLoading: false,
            varietyDebtLoading: false,
            transactionPanLoading: false,
            thirdPartyProductsLoading: false,
            indexList, // 指标列表
            assetsType: [],
            accountTreeData: [], // 账户树列表
            accountTreeLevelFourData: [], // 账户树四级
            categoriesOfAccounting: [],
            overviewData: [],   // 总览数据
            accountData: [],    // 分账户数据
            accountList: [],
            newConfigData: [],
            positionData: [],
            transactionData: [],
            varietyDebtData: [],
            transactionPanData: [],
            thirdPartyProductsData: [],
            aeDebounce: null,
            insureDate: this.sdate,
            markFlag: false,
            markTitle: '',
            isShow: true,
            formatterFlag: false,
            accountTreeLevelFourNewData: [], // 分账户的账户树
            showInfoFlag: false,
            mergeColumns: [],
            pos: 0
        };
    },
    watch: {
        positionActiveName(type) {
      // this.positionForm.date = this.sdate;
            this.loadPosition({ type: type, ...this.positionForm });
        },
        'overviewForm.date': function (val) {
            this.overviewSearch({ date: val }, 'overviewForm');
        }
    },
    methods: {
        initData(sdate) {
      // 页面最上的日期优先级最高
            this.getTips();
            this.loadIndices({ startDate: sdate });
            this.loadProfitOverview(this.overviewForm);
            this.loadNewConfiguration({ ...this.newConfigForm, startDate: this.newConfigForm.date[0], endDate: this.newConfigForm.date[1] });
            this.loadPosition({ type: this.positionActiveName, ...this.positionForm });
            this.loadTransaction({ ...this.transactionForm, startDate: this.transactionForm.date[0], endDate: this.transactionForm.date[1] });
            this.loadVarietyDebt({ startDate: sdate });
            this.loadTransactionpan({ startDate: sdate });
            this.loadThirdPartyProducts({ startDate: this.insureDate });
            this.loadCategoriesOfAccounting({sDate: sdate});
            this.loadAssetType();
            // 账户树，只加载一次
            //  this.accountTreeData.length < 1 && this.loadAccountTree({ department: 'FIX' });
            this.accountTreeLevelFourData < 1 && this.loadAccountTreeLevelFour({department: 'FIX'});
            this.accountTreeLevelFourNewData < 1 && this.loadAccountTreeLevelNewFour({department: 'FIX'});
        },
        insureDateChange() {
            this.loadThirdPartyProducts({ startDate: this.insureDate });
        },
        // ajax
        loadIndices(params) { // 加载主要指标
            this.indicesLoading = true;
            getIndices(params).then(({ data }) => {
                this.indices = data;
                this.indicesLoading = false;
            });
        },
        loadAccount(type) { // 加载账户列表
            this.accountDialogLoading = true;
            getAccount({ type }).then(({ data }) => {
                this.accountDialogLoading = false;
                this.accountList = data;
            });
        },
        loadProfitOverview(params) { // 加载总览
            this.overviewLoading = true;
            getProfitOverview(params).then(({ data }) => {
                this.overviewLoading = false;
                this.overviewData = data.rows;
            });
        },
        loadAccountTree(params) { // 获取账户树列表
            getAccountTree(params).then(({ data }) => {
                this.accountTreeData = data;
            });
        },
        loadAccountTreeLevelNewFour(params) { // 加载新账户树四级列表
            getAccountTreeNew(params).then(({data}) => {
                if (data) {
                    this.accountTreeLevelFourNewData = data;
                    this.showInfoFlag = true;
                } else {
                    this.showInfoFlag = false;
                }
            });
        },
        loadAccountTreeLevelFour(params) { // 加载账户树四级列表
            getAccountTreeLevelFour(params).then(({data}) => {
                this.accountTreeLevelFourData = data;
            });
        },
        loadProfitAccount(params) { // 加载分账户
            this.accountLoading = true;
            getProfitAccountNew(params).then(({ data }) => {
                console.log(data);
                this.accountData = data.rows;
                this.accountLoading = false;
            });
        },
        loadCategoriesOfAccounting(params) { // 获取会计分类
            Vue.set(params, 'type', 1);
            // console.log('params', params);
            let selectAll = [];
            getClassification(params).then(({ data }) => {
                this.categoriesOfAccounting = data;
                data.forEach(item => {
                   selectAll.push(item.combCode);
                });
                this.newConfigForm.categoriesOfAccounting = selectAll;
                this.transactionForm.categoriesOfAccounting = selectAll;
            });
        },
        loadNewConfiguration(params) { // 获取新增配置
            this.newConfigLoading = true;
            getNewConfiguration(params).then(({ data, error }) => {
                // console.log(data.rows);
                this.newConfigLoading = false;
                if (data) {
                    this.newConfigData = data.rows;
                } else {
                    this.newConfigData = [];
                    this.$message.error(error.message);
                }
            });
        },
        loadPosition(params) { // 加载持仓情况
            this.positionLoading = true;
            this.mergeColumns = [];
            getPosition(params).then(({ data }) => {
                this.positionLoading = false;
                if (this.positionActiveName === params.type) {
                    console.log(data.rows);
                    this.positionData = data.rows;
                    this.tableDataFilter(data.rows);
                }
            });
        },
        loadAssetType() { // 加载资产类别
            getAssetType({department: 'FIX'}).then(({ data }) => {
                this.assetsType = data;
                // console.log(this.assetsType);
            });
        },
        loadTransaction(params) { // 交易情况
            this.transactionLoading = true;
            getTransaction(params).then(({ data }) => {
                this.transactionLoading = false;
                this.transactionData = data.rows;
            });
        },
        loadVarietyDebt(params) { // 加载品种债剩余期限
            this.varietyDebtLoading = true;
            getVarietyDebt(params).then(({ data }) => {
                this.varietyDebtData = this.cellSpanData(data.rows);
                this.varietyDebtLoading = false;
            });
        },
        loadTransactionpan(params) { // 加载交易盘数据
            this.transactionPanLoading = true;
            getTransactionpan(params).then(({ data }) => {
                this.transactionPanData = data.rows;
                this.transactionPanLoading = false;
            });
        },
        loadThirdPartyProducts(params) { // 加载管理第三方产品情况数据
            this.thirdPartyProductsLoading = true;
            getThirdPartyProducts(params).then(({ data }) => {
                this.thirdPartyProductsData = data.rows;
                this.thirdPartyProductsLoading = false;
            });
        },
    // methods
        sdateChange(value) {
            console.log(22222);
            setDate('fixedIncome', value);
            this.initData(value);
            // 分账户查询
            this.loadProfitAccount({ date: this.sdate, ...this.accountForm });
            // sdate改变， 页面上所有的日期选择器都要改
            this.overviewForm.date = value;
            this.newConfigForm.date = [value, value];
            this.positionForm.date = value;
            this.transactionForm.date = [value, value];
            this.insureDate = value;
        },
        cellSpanData(data) { // 行合并数据处理
            const res = [];
            data.forEach(item => {
                item.forEach(cItem => {
                    cItem.span = item.length;
                    res.push(cItem);
                });
            });
            return res;
        },
        accountSelectClick(type) {
            this.accountVisible = true;
            this.activeAccountType = type;
      // 显示已选账户
            this.accountSelected = this.onSureAccount[type];
            this.loadAccount(type);
        },
        accountCancel() {
            this.accountVisible = false;
            this.accountSelected = [];
        },
    // 持仓 按账户&会计 品种债剩余期限  行合并
        spanMethod({ row, column, rowIndex, columnIndex }) {
            if (columnIndex === 0) {
                if (rowIndex % row.span === 0) {
                    return {
                        rowspan: row.span,
                        colspan: 1
                    };
                } else {
                    return {
                        rowspan: 0,
                        colspan: 0
                    };
                }
            }
        },
        positionByAllSpanMethod({ row, column, rowIndex, columnIndex }) {
            if (columnIndex === 0) {
                if (rowIndex % 5 === 0) {
                    return {
                        rowspan: 5,
                        colspan: 1
                    };
                } else {
                    return {
                        rowspan: 0,
                        colspan: 0
                    };
                }
            }
        },
        objectSpanMethod({ row, column, rowIndex, columnIndex }) {
            if (columnIndex === 0) {
                const _row = this.mergeColumns[rowIndex];
                const _col = 1;
                return {
                    rowspan: _row,
                    colspan: _col
                };
            }
        },
        tableDataFilter(data) {
            this.mergeColumns = [];
            for (let i = 0; i < data.length; i++) {
                if (i === 0) {
                    this.mergeColumns.push(1);
                    this.pos = 0;
                } else {
                    // 判断当前元素与上一个元素是否相同
                    if (data[i].account === data[i - 1].account) {
                        this.mergeColumns[this.pos] += 1;
                        this.mergeColumns.push(0);
                    } else {
                        this.mergeColumns.push(1);
                        this.pos = i;
                    }
                }
            }
        },
        cellStyle({ row, column, rowIndex, columnIndex }) {
            if (rowIndex % 2 !== 0 && columnIndex > 1) {
                if (row[column.property] > 0) {
                    return {
                        color: '#CC3D3D'
                    };
                } else {
                    return {
                        color: '#44B365'
                    };
                }
            }
        },
        debtFormatter(row, column, cellValue, index) {
            if (index % 4 !== 0) {
                return fixed2(cellValue) + '%';
            }
            return currencyFormatter(cellValue);
        },
        overviewSearch(formData, formName) { // 总览查询
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    this.loadProfitOverview(formData);
                } else {
                    return false;
                }
            });
        },
        overviewExport(formData, formName) { // 总览导出
            this.$refs['overviewForm'].validate((valid) => {
                if (valid) {
                    exportProfitOverview({date: this.overviewForm.date});
                } else {
                    return false;
                }
            });
        },
        accountSearch(formData) { // 分账户查询
            // 未选择账户不可查询
            if (this.accountForm.index === '规模占比' || this.accountForm.index === '实现收益率(减值后)' || this.accountForm.index === '综合收益率') {
                this.accountColumns.forEach((item, index) => {
                    if (index !== 0) {
                        item.render = percentFormatter;
                    }
                });
            } else {
                this.accountColumns.forEach((item, index) => {
                    if (index !== 0) {
                        item.render = currencyFormatter;
                    }
                });
            }
            if (this.accountForm.account.length < 1) {
                this.$message({
                    type: 'warning',
                    message: '请先选择账户'
                });
                return;
            }
            this.loadProfitAccount({ date: this.sdate, ...formData });
        },
        accountExport(formData) { // 分账户导出
            // 未选择账户不可查询
            if (this.accountForm.account.length < 1) {
                this.$message({
                    type: 'warning',
                    message: '请先选择账户'
                });
                return;
            }
            if (!this.aeDebounce) {
                this.aeDebounce = debounce(() => exportProfitAccount({ date: this.sdate, ...formData }), 15000, {
                    'leading': true,
                    'trailing': false
                });
            }
            this.aeDebounce();
        },
        newConfigSearch(formData, formName) { // 新增配置查询
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    const params = {
                        ...formData,
                        startDate: formData.date[0],
                        endDate: formData.date[1]
                    };
                    this.loadNewConfiguration(params);
                } else {
                    return false;
                }
            });
        },
        positionSearch(formData) {  // 持仓查询
            this.positionData = [];
            this.loadPosition({
                type: this.positionActiveName,
                ...formData });
            // console.log('loadPosition', this.loadPosition);
        },
        transactionSearch(formData, formName) { // 交易情况查询
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    const params = {
                        ...formData,
                        startDate: formData.date[0],
                        endDate: formData.date[1]
                    };
                    this.loadTransaction(params);
                }
            });
        },
        moreClick(moreBlock, moreBlockTitle) {
            if (moreBlock === 'thirdPartyProducts') {
                this.router_oms.push({
                    path: '/navigate/fixedIncomeMore',
                    query: { moreBlock, date: this.insureDate }
                });
            } else {
                this.router_oms.push({
                    path: '/navigate/fixedIncomeMore',
                    query: { moreBlock, date: this.sdate }
                });
            }
        },
        thousandthFormatter(row, column, cellValue, index) { // 金额千分位
            return currencyFormatter(cellValue);
        },
        percentFormatter(row, column, cellValue, index) {
            return fixed2(cellValue) + '%';
        },
        accountOnOk(nodes, formName) {
            this[formName].account = nodes;
            switch (formName) {
            case 'newConfigForm':
                this.newConfigSearch(this.newConfigForm, 'newConfigForm');
                break;
            }
        },
        assetsTypeOnOk(nodes, formName) {
            this[formName].productCategory = nodes;
        },
        accountAccountOnok(nodes, formName) {
            // let _self = this;
            if (this.accountForm.index === '规模占比' || this.accountForm.index === '实现收益率(减值后)' || this.accountForm.index === '综合收益率') {
                this.formatterFlag = true;
            } else {
                this.formatterFlag = false;
            }
            let accountNames = [];
            const accountIds = nodes.map(item => {
                accountNames.push(item.label);
                return item.id;
            });
            const accountColumns = nodes.map(item => {
                return {
                    label: item.label,
                    prop: item.id,
                    'min-width': 140,
                    'show-overflow-tooltip': true,
                    align: 'right',
                    render: this.formatterFlag ? percentFormatter : currencyFormatter
                };
            });
            this.accountColumns = this.accountColumns.slice(0, 1).concat(accountColumns);
            console.log(this.accountColumns);
            this[formName].account = accountIds.join(',');
            this[formName].accountNames = accountNames.join(',');
            this.accountSearch(this.accountForm);
        },
        /**
         * 下载功能
         * @param block 下载模块名
         * 完成
         */
        download(block) {
            switch (block) {
            case 'newConfiguration': // 新增配置
                let formData = this.newConfigForm;
                let params = {
                    ...formData,
                    startDate: formData.date[0],
                    endDate: formData.date[1]
                };
                exportNewConfiguration(params);
                break;
            case 'position':  // 持仓情况
                exportPosition({
                    type: this.positionActiveName, ...this.positionForm
                });
                break;
            case 'transaction':  // 交易情况
                formData = this.transactionForm;
                params = {
                    ...formData,
                    startDate: formData.date[0],
                    endDate: formData.date[1]
                };
                exportTransaction(params);
                break;
            case 'expectedAssets': // 存量资产预计到期情况
                formData = this.$refs.barOfExpectedAssets.form;
                params = {
                    ...formData,
                    startDate: formData.date[0],
                    endDate: formData.date[1]
                };
                exportExpectedAssets(params);
                break;
            case 'varietyDebt': // 品种债剩余期限
                exportVarietyDebt({startDate: this.sdate});
                break;
            case 'transactionpan': // 交易盘
                exportTransactionpan({startDate: this.sdate});
                break;
            case 'thirdPartyProducts': // 管理第三方产品情况
                exportThirdPartyProducts({startDate: this.insureDate});
                break;
            }
        },
        getTips() {
            let obj = {
                'sDate': this.sdate
            };
            getTips(obj).then((data) => {
                if (data.data) {
                    this.markFlag = true;
                    this.markTitle = data.data;
                    this.isShow = true;
                } else {
                    this.markFlag = false;
                    this.isShow = false;
                }
            });
        },
        colseMark(flag) {
            this.markFlag = flag;
        }
    },
    created() {
        const storageDate = getDate('fixedIncome');
        if (storageDate) {
            this.sdate = storageDate;
            this.overviewForm.date = this.sdate;
            this.newConfigForm.date = [this.sdate, this.sdate];
            this.positionForm.date = this.sdate;
            this.transactionForm.date = [this.sdate, this.sdate];
            this.insureDate = this.sdate;
            this.initData(this.sdate);
        } else {
            getWorkDay({date: newDate}).then(({data, error}) => {
                if (data) {
                    this.insureDate = data.date;
                    this.overviewForm.date = this.sdate;
                    this.newConfigForm.date = [this.sdate, this.sdate];
                    this.positionForm.date = this.sdate;
                    this.transactionForm.date = [this.sdate, this.sdate];
                    this.initData(this.sdate);
                }
            });
        }
    }
};
</script>

<style lang="scss" scoped>
.fixedIncome {
  position: relative;
  background-color: #f2f2f2;
}

.tab-bar {
    position: relative;
    .tab-right {
        position: absolute;
        top: 5px;
        right: 10px;
    }
}

// .searchbox {
//   position: fixed;
//   top: 54px;
//   right: 0;
//   z-index: 999;
//   background: #fff;
//   .right {
//     margin-right: 10px;
//   }
// }

.searchbox {
  background: #fff;
  padding: 5px 10px;
  border-bottom: 1px solid #e4e7ed;
  display: flex;
  justify-content: space-between;
  .left {
    margin-left: 10px;
  }
  .right {
    margin-right: 10px;
  }
}

.body {
  padding: 10px 0;
  > div {
    margin-bottom: 10px;
  }
  > section {
    margin-bottom: 20px;
    &:first-child {
      margin-top: 20px;
    }
    &:last-child {
      margin: 0;
    }
  }
}
.indicesCard {
  padding: 10px 0;
  box-sizing: border-box;
  height: 153px;
}

.accountDialog {
  .content {
    max-height: 400px;
    min-height: 100px;
    padding: 10px 20px;
    overflow: auto;
    border: 1px solid #dde3ea;
  }
  .accountDialogTitle {
    line-height: 32px;
    .title {
      float: left;
      font-size: 16px;
      color: #4b556a;
      font-weight: bold;
      i {
        font-size: 12px;
        font-style: normal;
        font-weight: 400;
        color: rgba(204, 61, 61, 0.85);
      }
    }
    .input {
      float: right;
      width: 240px;
      .el-input__icon {
        cursor: pointer;
        &:hover {
          color: #2464b2;
        }
      }
    }
  }
  .accountDialigCheckBox {
    .el-checkbox {
      margin-left: 30px;
    }
  }
}

.expectedOfStockAssets {
  .el-form {
    display: inline-block;
  }
  .totalNum {
    height: 33px;
    line-height: 33px;
    vertical-align: top;
    float: right;
    margin-right: 40px;
  }
}

.moreicon {
  width: 1.1em;
  height: 1.5em;
  cursor: pointer;
  + svg {
    margin-left: 10px; 
  }
  margin-bottom: -3px;
}
.clearfix:after {
  content: '.';
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
.clearfix {
  zoom: 1;
}
</style>


<style lang="scss">
@import '../tabs.scss';
@include self-tabs;
.fixedIncome {
  .spancolumn {
    vertical-align: top;
  }
  .el-table::before {
    height: 0;
  }
  .el-table__fixed::before,
  .el-table__fixed-right::before {
    height: 0;
  }
  .accountDialog {
    width: 80%;
    border-radius: 4px;
    .el-dialog__header {
      padding: 10px 20px;
    }
    .el-dialog__body {
      padding: 30px;
    }
    .el-dialog__footer {
      padding: 14px 20px;
      text-align: center;
    }
  }

  .accountDialog {
    .el-dialog__header {
      padding: 10px 20px;
    }
    .el-dialog__body {
      padding: 0;
      .el-collapse {
        border: none;
      }
      .el-collapse-item__header {
        border-bottom: none;
      }
    }
  }

  .profit {
    .el-tabs__nav {
      margin-left: 20px;
      width: 300px;
      .el-tabs__item.is-top {
        width: 120px;
        text-align: center;
      }
    }
  }

  .borderSetting {
      .el-table_6_column_89 {
          border-right: 1px solid #dbdcdc;
      }
  }

  .position {
     .el-tabs--border-card .el-tabs__nav .el-tabs__item.is-active {
        font-weight: 400;
        color: #2a76cd;
     }
  }

  .el-tabs--border-card > .el-tabs__content {
      padding: 10px 0;
  }
}
</style>

