<template>
  <div class="container">
    <table class="table">
      <thead>
        <tr>
          <th>거래일시</th>
          <th>기재내용</th>
          <th>지출</th>
          <th>수입</th>
          <th>잔액</th>
          <th>카테고리</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="item in sortedData"
          :key="item.no"
          @click="goToDetail(item.id)"
        >
          <td>{{ item.date }}</td>
          <td>{{ item.content }}</td>
          <td>{{ item.expense }}</td>
          <td>{{ item.income }}</td>
          <td>{{ item.balance }}</td>
          <td>{{ item.category }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  <div class="container">
    <div class="row">
      <button @click="addDataHandler">추가하기</button>
    </div>
    <div class="row">
      <table>
        <thead>
          <tr>
            <th></th>
            <th>거래일시</th>
            <th>기재내용</th>
            <th v-if="selectedType === 'expense'">지출</th>
            <th v-if="selectedType === 'income'">수입</th>
            <th>잔액</th>
            <th>카테고리</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>
              <label>수입</label>
              <input
                type="radio"
                name="income_expense"
                value="income"
                v-model="selectedType"
              />
              <label>지출</label>
              <input
                type="radio"
                name="income_expense"
                value="expense"
                v-model="selectedType"
              />
            </td>
            <td>
              <input
                type="text"
                v-model="dataItem.date"
                placeholder="yyyy.mm.dd hh:mm"
              />
            </td>
            <td>
              <input type="text" v-model="dataItem.content" />
            </td>
            <td v-if="selectedType === 'expense'">
              <input type="text" v-model="dataItem.expense" />
            </td>
            <td v-if="selectedType === 'income'">
              <input type="text" v-model="dataItem.income" />
            </td>
            <td>
              <input type="text" v-model="dataItem.balance" />
            </td>
            <td>
              <select v-model="dataItem.category">
                <optgroup v-if="selectedType === 'expense'">
                  <option>식비</option>
                  <option>교통</option>
                  <option>공과금</option>
                  <option>의료</option>
                  <option>경조사</option>
                  <option>금융</option>
                  <option>여가</option>
                </optgroup>
                <optgroup v-else-if="selectedType === 'income'">
                  <option>봉급</option>
                  <option>금융</option>
                  <option>용돈</option>
                  <option>기타</option>
                </optgroup>
              </select>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script setup>
import { useDataStore } from "@/stores/db.js";
import { ref, computed, reactive, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import moment from "moment";
import { storeToRefs } from "pinia";
// 데이터 불러오기
const dataStore = useDataStore();
const addData = dataStore.addData;
const router = useRouter();
const route = useRoute();
const convertToDate = (dateString) => {
  return moment(dateString, "YYYY.MM.DD HH:mm").toDate();
};

// 라우팅 함수
function goToDetail(itemId) {
  return router.push(`/Cash/edit/${itemId}`);
}

// 수입 지출 구분을 위한 변수 selectedType 선언
const selectedType = ref("income");
// Ensure data is initialized as an array
const sortedData = ref([]);
onMounted(async () => {
  await dataStore.requestAPI();
  sortedData.value = dataStore.data
    .map((item) => ({
      ...item,
      date: convertToDate(item.date),
    }))
    .sort(sortByDate);
});
// 날짜를 기준으로 객체를 정렬하는 비교 함수
const sortByDate = (a, b) => {
  return a.date - b.date; // 오름차순 정렬
  // return b.date - a.date; // 내림차순 정렬
};
// 추가를 위한 dataItem 정의
const dataItem = reactive({
  content: "",
  income: "",
  expense: "",
  category: "",
  balance: "",
  date: "",
});
//조건에 맞으면 데이터 추가해주는 함수인 addDataHandler함수 선언
const addDataHandler = () => {
  if (!dataItem.content || dataItem.content.trim() === "") {
    alert("내용을 입력해주세요!");
    return;
  }
  if (!dataItem.expense && !dataItem.income) {
    alert("금액을 입력해주세요!");
    return;
  }
  if (isNaN(dataItem.expense) || isNaN(dataItem.income)) {
    // 변경된 부분
    alert("금액은 숫자로 입력해주세요!"); // 변경된 부분
    return;
  }
  if (!dataItem.category) {
    alert("카테고리를 선택해주세요!");
    return;
  }
  if (!dataItem.date) {
    alert("날짜를 입력해주세요!");
    return;
  }
  addData({ ...dataItem }, async () => {
    await dataStore.requestAPI();
    sortedData.value = dataStore.data
      .map((item) => ({
        ...item,
        date: convertToDate(item.date),
      }))
      .sort(sortByDate);
  });
};
</script>
