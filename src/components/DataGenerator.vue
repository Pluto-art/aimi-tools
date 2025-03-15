<script setup>
import { ref, computed, watch } from 'vue';

// 存储生成的数据集合
const generatedData = ref([]);
const dataCount = ref(4); // 默认生成8条数据
const copyMessage = ref(''); // 复制提示消息
let copyTimer = null; // 复制提示计时器

// 银行卡数据
const banks = [
  { name: '中国工商银行', bins: ['622200', '622202', '622203', '622208', '621226', '402791'] },
  { name: '中国农业银行', bins: ['622848', '622700', '622400', '621288', '622836', '622820'] },
  { name: '中国银行', bins: ['622262', '622260', '621660', '622690', '622760', '622752'] },
  { name: '中国建设银行', bins: ['622280', '622700', '622707', '622966', '622988', '621700'] },
  { name: '交通银行', bins: ['622260', '622250', '622298', '622908', '622918', '622286'] },
  { name: '招商银行', bins: ['622588', '622575', '622576', '622579', '622581', '622582'] },
  { name: '中国邮政储蓄银行', bins: ['622188', '622150', '622151', '622181', '622200', '621095'] },
  { name: '中信银行', bins: ['622611', '622600', '622656', '622610', '622637', '621050'] },
  { name: '华夏银行', bins: ['622630', '622631', '622632', '622633', '603506', '621021'] },
  { name: '中国民生银行', bins: ['622622', '622617', '622615', '622616', '622618', '622620'] },
  { name: '广发银行', bins: ['622568', '622569', '622578', '622575', '622576', '622577'] },
  { name: '平安银行', bins: ['622155', '622156', '622157', '622158', '622159', '621351'] },
  { name: '兴业银行', bins: ['622909', '622908', '622516', '622517', '622518', '622939'] },
  { name: '浦发银行', bins: ['622522', '622525', '622509', '622538', '622528', '622520'] }
];

// 省份数据
const provinces = [
  { name: '北京市', codes: ['110100', '110200'] },
  { name: '上海市', codes: ['310100', '310200'] },
  { name: '天津市', codes: ['120100', '120200'] },
  { name: '重庆市', codes: ['500100', '500200'] },
  { name: '河北省', codes: ['130100', '130200', '130300', '130400', '130500', '130600'] },
  { name: '山西省', codes: ['140100', '140200', '140300', '140400', '140500'] },
  { name: '辽宁省', codes: ['210100', '210200', '210300', '210400'] },
  { name: '吉林省', codes: ['220100', '220200', '220300', '220400'] },
  { name: '黑龙江省', codes: ['230100', '230200', '230300', '230400'] },
  { name: '江苏省', codes: ['320100', '320200', '320300', '320400', '320500'] },
  { name: '浙江省', codes: ['330100', '330200', '330300', '330400'] },
  { name: '安徽省', codes: ['340100', '340200', '340300', '340400'] },
  { name: '福建省', codes: ['350100', '350200', '350300', '350400'] },
  { name: '江西省', codes: ['360100', '360200', '360300', '360400'] },
  { name: '山东省', codes: ['370100', '370200', '370300', '370400', '370500'] },
  { name: '河南省', codes: ['410100', '410200', '410300', '410400'] },
  { name: '湖北省', codes: ['420100', '420200', '420300', '420400'] },
  { name: '湖南省', codes: ['430100', '430200', '430300', '430400'] },
  { name: '广东省', codes: ['440100', '440200', '440300', '440400'] },
  { name: '广西壮族自治区', codes: ['450100', '450200', '450300'] },
  { name: '海南省', codes: ['460100', '460200'] },
  { name: '四川省', codes: ['510100', '510200', '510300', '510400'] },
  { name: '贵州省', codes: ['520100', '520200', '520300'] },
  { name: '云南省', codes: ['530100', '530200', '530300'] },
  { name: '陕西省', codes: ['610100', '610200', '610300', '610400'] },
  { name: '甘肃省', codes: ['620100', '620200', '620300'] },
  { name: '青海省', codes: ['630100', '630200'] },
  { name: '内蒙古自治区', codes: ['150100', '150200', '150300'] },
  { name: '西藏自治区', codes: ['540100', '540200'] },
  { name: '宁夏回族自治区', codes: ['640100', '640200'] },
  { name: '新疆维吾尔自治区', codes: ['650100', '650200', '650300'] }
];

// 默认选择北京市
const selectedProvince = ref(provinces[0]);

// 获取当前年份计算18岁
const currentYear = new Date().getFullYear();
const adultYear = currentYear - 18;

// 出生日期选择器 (默认为18岁)
const birthYear = ref(adultYear); // 默认18岁
const birthMonth = ref(1);   // 默认月份
const birthDay = ref(1);     // 默认日期

// 可选年份范围
const yearOptions = computed(() => {
  const years = [];
  for (let year = 1950; year <= currentYear; year++) {
    years.push(year);
  }
  return years;
});

// 可选月份
const monthOptions = computed(() => {
  const months = [];
  for (let month = 1; month <= 12; month++) {
    months.push(month);
  }
  return months;
});

// 当前月份的最大天数
const daysInMonth = computed(() => {
  if ([1, 3, 5, 7, 8, 10, 12].includes(birthMonth.value)) {
    return 31;
  } else if ([4, 6, 9, 11].includes(birthMonth.value)) {
    return 30;
  } else {
    // 2月，考虑闰年
    const isLeapYear = (birthYear.value % 4 === 0 && birthYear.value % 100 !== 0) || (birthYear.value % 400 === 0);
    return isLeapYear ? 29 : 28;
  }
});

// 可选日期
const dayOptions = computed(() => {
  const days = [];
  for (let day = 1; day <= daysInMonth.value; day++) {
    days.push(day);
  }
  return days;
});

// 监听月份变化，确保日期有效
watch([birthYear, birthMonth], () => {
  if (birthDay.value > daysInMonth.value) {
    birthDay.value = daysInMonth.value;
  }
});

// 常见中国姓氏数组
const familyNames = [
  '李', '王', '张', '刘', '陈', '杨', '赵', '黄', '周', '吴',
  '徐', '孙', '胡', '朱', '高', '林', '何', '郭', '马', '罗',
  '梁', '宋', '郑', '谢', '韩', '唐', '冯', '于', '董', '萧',
  '程', '曹', '袁', '邓', '许', '傅', '沈', '曾', '彭', '吕'
];

// 生成中文名字的常见汉字
const nameChars = [
  '伟', '芳', '娜', '秀英', '敏', '静', '丽', '强', '磊', '军',
  '洋', '勇', '艳', '杰', '娟', '涛', '明', '超', '秀兰', '霞',
  '平', '刚', '桂英', '世', '华', '辉', '亮', '兰', '凯', '健',
  '柳', '子', '文', '江', '梅', '鹏', '志', '宏', '丹', '丽华'
];

// 生成随机整数
function getRandomInt(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// 生成随机姓名
function generateName() {
  const familyName = familyNames[Math.floor(Math.random() * familyNames.length)];
  const givenName = nameChars[Math.floor(Math.random() * nameChars.length)];
  return familyName + givenName;
}

// 生成随机手机号码
function generatePhone() {
  // 手机号前三位
  const prefixes = [
    '130', '131', '132', '133', '134', '135', '136', '137', '138', '139',
    '150', '151', '152', '153', '155', '156', '157', '158', '159',
    '166', '176', '177', '178',
    '180', '181', '182', '183', '184', '185', '186', '187', '188', '189',
    '198', '199'
  ];
  
  const prefix = prefixes[Math.floor(Math.random() * prefixes.length)];
  let rest = '';
  
  for (let i = 0; i < 8; i++) {
    rest += Math.floor(Math.random() * 10);
  }
  
  return prefix + rest;
}

// 生成随机身份证号
function generateIdCard() {
  // 从选定省份的地区码中随机选择一个
  const provinceCodes = selectedProvince.value.codes;
  const areaCode = provinceCodes[Math.floor(Math.random() * provinceCodes.length)];
  
  // 使用选择的出生日期
  const year = birthYear.value;
  const month = birthMonth.value;
  const day = birthDay.value;
  
  // 格式化日期为字符串
  const yearStr = year.toString();
  const monthStr = month.toString().padStart(2, '0');
  const dayStr = day.toString().padStart(2, '0');
  
  const birthday = `${yearStr}${monthStr}${dayStr}`;
  
  // 顺序码
  const sequenceCode = getRandomInt(1, 999).toString().padStart(3, '0');
  
  // 前17位
  const idCardBase = areaCode + birthday + sequenceCode;
  
  // 计算校验码
  const weights = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2];
  const checkCodes = ['1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'];
  
  let sum = 0;
  for (let i = 0; i < idCardBase.length; i++) {
    sum += parseInt(idCardBase[i]) * weights[i];
  }
  
  const checkCode = checkCodes[sum % 11];
  
  return idCardBase + checkCode;
}

// 生成银行卡号
function generateBankCard() {
  // 随机选择一个银行
  const bank = banks[Math.floor(Math.random() * banks.length)];
  
  // 随机选择该银行的一个BIN码
  const bin = bank.bins[Math.floor(Math.random() * bank.bins.length)];
  
  // 卡号长度，大多数银行卡为16位
  const length = 16;
  
  // 生成随机中间号码
  let cardNumber = bin;
  const middleLength = length - bin.length - 1; // 减1是为了留出校验位
  
  for (let i = 0; i < middleLength; i++) {
    cardNumber += Math.floor(Math.random() * 10);
  }
  
  // 使用Luhn算法计算校验位
  const checkDigit = calculateLuhnCheckDigit(cardNumber);
  
  // 返回完整卡号和银行名称
  return {
    number: cardNumber + checkDigit,
    bank: bank.name
  };
}

// Luhn算法计算校验位
function calculateLuhnCheckDigit(partialCardNumber) {
  let sum = 0;
  let alternate = false;
  
  // 从右向左处理每一位数字
  for (let i = partialCardNumber.length - 1; i >= 0; i--) {
    let n = parseInt(partialCardNumber[i], 10);
    
    if (alternate) {
      n *= 2;
      if (n > 9) {
        n = (n % 10) + 1;
      }
    }
    
    sum += n;
    alternate = !alternate;
  }
  
  // 计算校验位，使总和能被10整除
  return ((10 - (sum % 10)) % 10).toString();
}

// 脱敏处理函数
function maskName(name) {
  if (!name || name.length <= 1) return name;
  return name.charAt(0) + '*'.repeat(name.length - 1);
}

function maskPhone(phone) {
  if (!phone || phone.length < 11) return phone;
  return phone.substring(0, 3) + '****' + phone.substring(7);
}

function maskIdCard(idCard) {
  if (!idCard || idCard.length < 18) return idCard;
  return idCard.substring(0, 4) + '**********' + idCard.substring(14);
}

function maskBankCard(cardNumber) {
  if (!cardNumber || cardNumber.length < 10) return cardNumber;
  return cardNumber.substring(0, 4) + ' **** **** ' + cardNumber.substring(cardNumber.length - 4);
}

// 生成单条数据
function generateSingleData() {
  const bankCard = generateBankCard();
  
  return {
    name: generateName(),
    phone: generatePhone(),
    idCard: generateIdCard(),
    bankCard: bankCard.number,
    bankName: bankCard.bank
  };
}

// 生成多条数据
function generateMultipleData() {
  generatedData.value = [];
  for (let i = 0; i < dataCount.value; i++) {
    generatedData.value.push(generateSingleData());
  }
}

// 复制文本到剪贴板
function copyToClipboard(text) {
  navigator.clipboard.writeText(text).then(() => {
    showCopyMessage('复制成功！');
  }).catch(err => {
    showCopyMessage('复制失败，请手动复制');
    console.error('复制失败:', err);
  });
}

// 显示复制提示信息
function showCopyMessage(message) {
  copyMessage.value = message;
  
  // 清除之前的计时器
  if (copyTimer) {
    clearTimeout(copyTimer);
  }
  
  // 设置新的计时器，2秒后隐藏消息
  copyTimer = setTimeout(() => {
    copyMessage.value = '';
  }, 2000);
}

// 初始生成数据
generateMultipleData();
</script>

<template>
  <div class="data-generator">
 
    
    <div class="disclaimer">
      <p>📢 <strong>免责声明：</strong>本工具生成的所有数据仅用于系统测试和开发调试目的，不代表任何真实个人信息，如有雷同，纯属巧合。严禁将生成数据用于非法用途，使用者应当遵守相关法律法规。</p>
    </div>
    
    <div class="controls">
      <div class="settings-row">
        <div class="count-control">
          <label for="data-count">生成数量：</label>
          <input 
            id="data-count" 
            type="number" 
            min="1" 
            max="100" 
            v-model="dataCount"
          />
        </div>
        
        <div class="province-selector">
          <label for="province-select">身份证省份：</label>
          <select 
            id="province-select" 
            v-model="selectedProvince"
          >
            <option v-for="province in provinces" :key="province.name" :value="province">
              {{ province.name }}
            </option>
          </select>
        </div>
        
        <div class="birth-settings">
          <div class="birth-selector-row">
            <div class="birth-title">身份证出生日期：</div>
            <div class="birth-selectors">
              <div class="birth-control">
                <select id="birth-year" v-model="birthYear">
                  <option v-for="year in yearOptions" :key="year" :value="year">{{ year }}年</option>
                </select>
              </div>
              
              <div class="birth-control">
                <select id="birth-month" v-model="birthMonth">
                  <option v-for="month in monthOptions" :key="month" :value="month">{{ month }}月</option>
                </select>
              </div>
              
              <div class="birth-control">
                <select id="birth-day" v-model="birthDay">
                  <option v-for="day in dayOptions" :key="day" :value="day">{{ day }}日</option>
                </select>
              </div>
            </div>
          </div>
        </div>
        
        <button class="generate-all" @click="generateMultipleData">生成测试数据</button>
      </div>
    </div>
    
    <div class="copy-message" v-if="copyMessage">{{ copyMessage }}</div>
    
    <div class="data-cards">
      <div v-for="(item, index) in generatedData" :key="index" class="data-card">
        <div class="card-header">
          <span class="card-title">测试数据 #{{ index + 1 }}</span>
        </div>
        <div class="card-body">
          <div class="data-row">
            <span class="data-label">姓名：</span>
            <span class="data-value" @click="copyToClipboard(item.name)" title="点击复制原始数据">
              {{ maskName(item.name) }}
              <span class="copy-icon">📋</span>
            </span>
          </div>
          
          <div class="data-row">
            <span class="data-label">手机号：</span>
            <span class="data-value" @click="copyToClipboard(item.phone)" title="点击复制原始数据">
              {{ maskPhone(item.phone) }}
              <span class="copy-icon">📋</span>
            </span>
          </div>
          
          <div class="data-row">
            <span class="data-label">身份证号：</span>
            <span class="data-value" @click="copyToClipboard(item.idCard)" title="点击复制原始数据">
              {{ maskIdCard(item.idCard) }}
              <span class="copy-icon">📋</span>
            </span>
          </div>
          
          <div class="data-row">
            <span class="data-label">银行卡：</span>
            <span class="data-value" @click="copyToClipboard(item.bankCard)" title="点击复制原始数据">
              {{ maskBankCard(item.bankCard) }}
              <span class="copy-icon">📋</span>
            </span>
          </div>
          
          <div class="data-row">
            <span class="data-label">银行名称：</span>
            <span class="data-value" @click="copyToClipboard(item.bankName)" title="点击复制原始数据">
              {{ item.bankName }}
              <span class="copy-icon">📋</span>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.data-generator {
  width: 100%;
  max-width: 1600px;
  margin: 0 auto;
  padding: 1rem;
}

.controls {
  margin-bottom: 1.5rem;
}

.settings-row {
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
  align-items: center;
}

.count-control {
  display: flex;
  align-items: center;
}

.count-control input {
  width: 60px;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  text-align: center;
  font-size: 1rem;
  background-color: rgba(255, 255, 255, 0.1);
  color: inherit;
}

.province-selector {
  display: flex;
  align-items: center;
  min-width: 250px;
}

.birth-settings {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.birth-selector-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.birth-title {
  font-weight: bold;
  white-space: nowrap;
}

.count-control, .province-control, .birth-control {
  display: flex;
  align-items: center;
}

.province-selector label, .count-control label, .birth-control label {
  margin-right: 0.5rem;
  font-weight: bold;
  white-space: nowrap;
}

.count-control input {
  width: 60px;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  text-align: center;
  font-size: 1rem;
  background-color: rgba(255, 255, 255, 0.1);
  color: inherit;
}

.province-selector select, .birth-control select {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
  min-width: 100px;
  background-color: rgba(255, 255, 255, 0.1);
  color: inherit;
  cursor: pointer;
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  background-image: url("data:image/svg+xml;utf8,<svg fill='gray' height='24' viewBox='0 0 24 24' width='24' xmlns='http://www.w3.org/2000/svg'><path d='M7 10l5 5 5-5z'/><path d='M0 0h24v24H0z' fill='none'/></svg>");
  background-repeat: no-repeat;
  background-position: right 8px center;
  padding-right: 30px;
}

.province-selector select {
  min-width: 150px;
}

.birth-selectors {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: center;
}

.generate-all {
  font-size: 1rem;
  padding: 0.7rem 1.5rem;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
  margin-top: 0;
  min-width: 150px;
  height: fit-content;
}

.generate-all:hover {
  background-color: #3ca576;
}

.copy-message {
  position: fixed;
  top: 20px;
  right: 20px;
  background-color: rgba(66, 184, 131, 0.9);
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.data-cards {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 固定4列 */
  gap: 1rem;
}

.data-card {
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s, box-shadow 0.2s;
  min-width: 350px;
}

.data-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.card-header {
  background-color: #42b883;
  color: white;
  padding: 0.8rem;
  font-weight: bold;
}

.card-body {
  padding: 1.2rem;
}

.data-row {
  display: flex;
  margin-bottom: 0.8rem;
  align-items: center;
  flex-wrap: wrap;
}

.data-row:last-child {
  margin-bottom: 0;
}

.data-label {
  width: 100px;
  font-weight: bold;
}

.data-value {
  flex: 1;
  font-family: monospace;
  background: rgba(0, 0, 0, 0.1);
  padding: 0.7rem;
  border-radius: 4px;
  word-break: break-all;
  cursor: pointer;
  position: relative;
  transition: background-color 0.2s;
}

.data-value:hover {
  background: rgba(66, 184, 131, 0.2);
}

.copy-icon {
  position: absolute;
  right: 5px;
  top: 5px;
  font-size: 14px;
  opacity: 0;
  transition: opacity 0.2s;
}

.data-value:hover .copy-icon {
  opacity: 1;
}

/* 响应式布局 */
@media (max-width: 1200px) {
  .data-cards {
    grid-template-columns: repeat(3, 1fr); /* 三列 */
  }
  
  .settings-row {
    gap: 0.8rem;
  }
}

@media (max-width: 900px) {
  .settings-row {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }
  
  .count-control, .province-selector, .birth-settings, .generate-all {
    width: 100%;
    max-width: 500px;
  }
  
  .count-control {
    justify-content: space-between;
  }
  
  .generate-all {
    margin-top: 0.5rem;
  }
}

@media (max-width: 600px) {
  .data-cards {
    grid-template-columns: repeat(1, 1fr); /* 单列 */
  }
  
  .birth-selector-row {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .birth-title {
    margin-bottom: 0.5rem;
  }
  
  .birth-selectors {
    flex-direction: column;
    align-items: stretch;
  }
  
  .birth-control {
    width: 100%;
    margin-bottom: 0.5rem;
  }
  
  .birth-control select {
    width: 100%;
  }
  
  .province-selector {
    flex-direction: column;
    align-items: stretch;
  }
  
  .province-selector label {
    margin-bottom: 0.3rem;
  }
  
  .province-selector select {
    width: 100%;
  }
}

.disclaimer {
  background-color: rgba(255, 229, 100, 0.3);
  border-left: 4px solid #ffe564;
  border-radius: 4px;
  margin: 1rem 0 1.5rem;
  padding: 0.8rem 1rem;
}

.disclaimer p {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
  color: rgba(0, 0, 0, 0.7);
}
</style> 