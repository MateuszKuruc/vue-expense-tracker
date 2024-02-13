<template>
  <Header />
  <div class="container">
    <Balance :total="+total" />
    <IncomeExpenses :income="+income" :expenses="-expenses" />
    <TransactionList
      :transactions="transactions"
      @transactionDeleted="deleteTransaction"
    />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import Header from "./components/Header.vue";
import Balance from "./components/Balance.vue";
import IncomeExpenses from "./components/IncomeExpenses.vue";
import TransactionList from "./components/TransactionList.vue";
import AddTransaction from "./components/AddTransaction.vue";

import { useToast } from "vue-toastification";

import { ref, computed, onMounted } from "vue";

onMounted(() => {
  const savedTransactions = JSON.parse(
    localStorage.getItem("transactionsData")
  );

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

const saveToLocalStorage = () => {
  localStorage.setItem("transactionsData", JSON.stringify(transactions.value));
};

const toast = useToast();

const transactions = ref([]);

// get total
const total = computed(() => {
  return transactions.value
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

// get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0)
    .toFixed(2);
});

const handleTransactionSubmitted = (transactionData) => {
  const newTransaction = {
    id: transactions.value.length + 1,
    text: transactionData.text,
    amount: transactionData.amount,
  };

  transactions.value.push(newTransaction);

  saveToLocalStorage();

  toast.success("Transaction added successfully");
};

const deleteTransaction = (id) => {
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );

  saveToLocalStorage();

  toast.success("Transaction deleted successfully");
};
</script>
