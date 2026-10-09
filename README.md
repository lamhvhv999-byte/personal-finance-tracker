const STORAGE_KEY = 'personal-finance-tracker-v1';

const categoryMap = {
  income: ['Lương', 'Freelance', 'Thưởng', 'Đầu tư', 'Khác'],
  expense: ['Ăn uống', 'Nhà ở', 'Đi lại', 'Hóa đơn', 'Giải trí', 'Y tế', 'Mua sắm', 'Học tập', 'Du lịch', 'Khác']
};

const palette = ['#2563eb', '#10b981', '#f59e0b', '#8b5cf6', '#ef4444', '#14b8a6', '#f97316', '#0ea5e9', '#22c55e', '#a855f7'];

const sampleTransactions = [
  { id: crypto.randomUUID(), date: '2026-10-01', type: 'income', category: 'Lương', amount: 22000000, note: 'Lương tháng 10' },
  { id: crypto.randomUUID(), date: '2026-10-03', type: 'expense', category: 'Ăn uống', amount: 850000, note: 'Ăn trưa công ty' },
  { id: crypto.randomUUID(), date: '2026-10-05', type: 'expense', category: 'Đi lại', amount: 420000, note: 'Xăng xe' },
  { id: crypto.randomUUID(), date: '2026-10-08', type: 'expense', category: 'Nhà ở', amount: 5500000, note: 'Tiền nhà' },
  { id: crypto.randomUUID(), date: '2026-10-12', type: 'income', category: 'Freelance', amount: 3500000, note: 'Dự án phụ' },
  { id: crypto.randomUUID(), date: '2026-10-18', type: 'expense', category: 'Giải trí', amount: 700000, note: 'Đi xem phim' },
  { id: crypto.randomUUID(), date: '2026-10-20', type: 'expense', category: 'Mua sắm', amount: 1500000, note: 'Mua đồ gia dụng' },
  { id: crypto.randomUUID(), date: '2026-10-25', type: 'expense', category: 'Y tế', amount: 900000, note: 'Khám sức khỏe' }
];

const state = {
  transactions: loadTransactions(),
  filterMonth: getCurrentMonth(),
  filterType: 'all',
  editingId: null
};

const elements = {
  monthFilter: document.getElementById('monthFilter'),
  typeFilter: document.getElementById('typeFilter'),
  incomeStat: document.getElementById('incomeStat'),
  expenseStat: document.getElementById('expenseStat'),
  balanceStat: document.getElementById('balanceStat'),
  savingStat: document.getElementById('savingStat'),
  budgetValue: document.getElementById('budgetValue'),
  budgetStatus: document.getElementById('budgetStatus'),
  budgetProgressBar: document.getElementById('budgetProgressBar'),
  transactionTableBody: document.getElementById('transactionTableBody'),
  transactionCount: document.getElementById('transactionCount'),
  categoryBreakdown: document.getElementById('categoryBreakdown'),
  formTitle: document.getElementById('formTitle'),
  transactionForm: document.getElementById('transactionForm'),
  transactionId: document.getElementById('transactionId'),
  dateInput: document.getElementById('dateInput'),
  typeInput: document.getElementById('typeInput'),
  categoryInput: document.getElementById('categoryInput'),
  amountInput: document.getElementById('amountInput'),
  noteInput: document.getElementById('noteInput'),
  cancelEditBtn: document.getElementById('cancelEditBtn'),
  resetDataBtn: document.getElementById('resetDataBtn')
};

init();

function init() {
  populateMonthFilter();
  setupEventListeners();
  refreshCategoryOptions();
  resetForm();
  render();
}

function setupEventListeners() {
  elements.typeFilter.addEventListener('change', (event) => {
    state.filterType = event.target.value;
    render();
  });

  elements.monthFilter.addEventListener('change', (event) => {
    state.filterMonth = event.target.value;
    render();
  });

  elements.typeInput.addEventListener('change', () => {
    refreshCategoryOptions();
  });

  elements.transactionForm.addEventListener('submit', (event) => {
    event.preventDefault();
    const payload = {
      id: elements.transactionId.value || crypto.randomUUID(),
      date: elements.dateInput.value,
      type: elements.typeInput.value,
      category: elements.categoryInput.value,
      amount: Number(elements.amountInput.value),
      note: elements.noteInput.value.trim() || 'Không có ghi chú'
    };

    if (!payload.date || !payload.category || !payload.amount) {
      return;
    }

    const existingIndex = state.transactions.findIndex((item) => item.id === payload.id);

    if (existingIndex >= 0) {
      state.transactions[existingIndex] = payload;
    } else {
      state.transactions.unshift(payload);
    }

    saveTransactions();
    resetForm();
    render();
  });

  elements.cancelEditBtn.addEventListener('click', () => {
    resetForm();
  });

  elements.resetDataBtn.addEventListener('click', () => {
    state.transactions = sampleTransactions.map((item) => ({ ...item, id: crypto.randomUUID() }));
    saveTransactions();
    state.filterMonth = getCurrentMonth();
    elements.monthFilter.value = state.filterMonth;
    elements.typeFilter.value = 'all';
    state.filterType = 'all';
    resetForm();
    render();
  });
}

function populateMonthFilter() {
  const months = new Set();
  state.transactions.forEach((item) => {
    if (item.date) months.add(item.date.slice(0, 7));
  });

  const allMonths = [...months].sort().reverse();
  const currentMonth = getCurrentMonth();

  if (!allMonths.includes(currentMonth)) {
    allMonths.unshift(currentMonth);
  }

  elements.monthFilter.innerHTML = allMonths
    .map((month) => `<option value="${month}">${formatMonthLabel(month)}</option>`)
    .join('');

  state.filterMonth = allMonths.includes(state.filterMonth) ? state.filterMonth : currentMonth;
  elements.monthFilter.value = state.filterMonth;
}

function loadTransactions() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) {
      return sampleTransactions;
    }

    const parsed = JSON.parse(saved);
    if (!Array.isArray(parsed) || parsed.length === 0) {
      return sampleTransactions;
    }

    return parsed;
  } catch (error) {
    return sampleTransactions;
  }
}

function saveTransactions() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state.transactions));
  populateMonthFilter();
}

function getCurrentMonth() {
  return new Date().toISOString().slice(0, 7);
}

function formatMonthLabel(monthValue) {
  const [year, month] = monthValue.split('-');
  const date = new Date(Number(year), Number(month) - 1, 1);
  return `${date.toLocaleString('vi-VN', { month: 'long' })} ${year}`;
}

function refreshCategoryOptions() {
  const type = elements.typeInput.value;
  const options = categoryMap[type] || categoryMap.expense;

  elements.categoryInput.innerHTML = options
    .map((category) => `<option value="${category}">${category}</option>`)
    .join('');
}

function render() {
  const filteredTransactions = getFilteredTransactions();
  const summary = summarizeTransactions(filteredTransactions);

  renderSummary(summary);
  renderBreakdown(filteredTransactions);
  renderTable(filteredTransactions);
}

function getFilteredTransactions() {
  return state.transactions.filter((item) => {
    const matchesMonth = item.date.startsWith(state.filterMonth || getCurrentMonth());
    const matchesType = state.filterType === 'all' || item.type === state.filterType;
    return matchesMonth && matchesType;
  });
}

function summarizeTransactions(transactions) {
  const income = transactions
    .filter((item) => item.type === 'income')
    .reduce((sum, item) => sum + Number(item.amount || 0), 0);

  const expense = transactions
    .filter((item) => item.type === 'expense')
    .reduce((sum, item) => sum + Number(item.amount || 0), 0);

  const balance = income - expense;
  const savings = Math.max(balance, 0);
  const monthlyBudget = 25000000;
  const usedBudget = expense > monthlyBudget ? monthlyBudget : expense;
  const budgetPercent = Math.min((usedBudget / monthlyBudget) * 100, 100);

  return { income, expense, balance, savings, monthlyBudget, budgetPercent };
}

function renderSummary(summary) {
  elements.incomeStat.textContent = formatCurrency(summary.income);
  elements.expenseStat.textContent = formatCurrency(summary.expense);
  elements.balanceStat.textContent = formatCurrency(summary.balance);
  elements.savingStat.textContent = formatCurrency(summary.savings);

  elements.budgetValue.textContent = formatCurrency(summary.monthlyBudget);
  elements.budgetStatus.textContent = `${Math.round(summary.budgetPercent)}% đã sử dụng`;
  elements.budgetProgressBar.style.width = `${summary.budgetPercent}%`;
}

function renderBreakdown(transactions) {
  const expenseByCategory = transactions
    .filter((item) => item.type === 'expense')
    .reduce((map, item) => {
      map[item.category] = (map[item.category] || 0) + Number(item.amount || 0);
      return map;
    }, {});

  const entries = Object.entries(expenseByCategory).sort(([, a], [, b]) => b - a);
  const maxValue = entries[0]?.[1] || 1;

  if (!entries.length) {
    elements.categoryBreakdown.innerHTML = '<div class="empty-state">Chưa có dữ liệu chi tiêu trong tháng này.</div>';
    return;
  }

  elements.categoryBreakdown.innerHTML = entries
    .slice(0, 6)
    .map(([category, amount], index) => {
      const width = (amount / maxValue) * 100;
      return `
        <div class="category-row">
          <div class="meta">
            <span class="dot" style="background:${palette[index % palette.length]}"></span>
            <span class="category-name">${category}</span>
          </div>
          <strong>${formatCurrency(amount)}</strong>
          <div class="category-bar-wrap">
            <span class="category-bar" style="width:${width}%; background:${palette[index % palette.length]};"></span>
          </div>
        </div>
      `;
    })
    .join('');
}

function renderTable(transactions) {
  if (!transactions.length) {
    elements.transactionTableBody.innerHTML = `
      <tr>
        <td colspan="6" class="empty-state">Không có giao dịch nào phù hợp.</td>
      </tr>
    `;
    elements.transactionCount.textContent = '0 mục';
    return;
  }

  elements.transactionCount.textContent = `${transactions.length} mục`;

  elements.transactionTableBody.innerHTML = transactions
    .sort((a, b) => new Date(b.date) - new Date(a.date))
    .map((item) => `
      <tr>
        <td>${formatDate(item.date)}</td>
        <td>
          <span class="type-pill ${item.type}">${item.type === 'income' ? 'Thu nhập' : 'Chi tiêu'}</span>
        </td>
        <td>${item.category}</td>
        <td>${item.note}</td>
        <td class="amount ${item.type}">${item.type === 'income' ? '+' : '-'} ${formatCurrency(item.amount)}</td>
        <td>
          <div class="row-actions">
            <button class="icon-btn edit" data-action="edit" data-id="${item.id}">Sửa</button>
            <button class="icon-btn delete" data-action="delete" data-id="${item.id}">Xóa</button>
          </div>
        </td>
      </tr>
    `)
    .join('');

  elements.transactionTableBody.querySelectorAll('[data-action="edit"]').forEach((button) => {
    button.addEventListener('click', (event) => {
      const { id } = event.currentTarget.dataset;
      const item = state.transactions.find((entry) => entry.id === id);
      if (!item) return;

      state.editingId = id;
      elements.formTitle.textContent = 'Sửa giao dịch';
      elements.transactionId.value = item.id;
      elements.dateInput.value = item.date;
      elements.typeInput.value = item.type;
      refreshCategoryOptions();
      elements.categoryInput.value = item.category;
      elements.amountInput.value = item.amount;
      elements.noteInput.value = item.note;
      elements.cancelEditBtn.classList.remove('hidden');
      elements.dateInput.focus();
    });
  });

  elements.transactionTableBody.querySelectorAll('[data-action="delete"]').forEach((button) => {
    button.addEventListener('click', (event) => {
      const { id } = event.currentTarget.dataset;
      state.transactions = state.transactions.filter((item) => item.id !== id);
      saveTransactions();
      if (state.editingId === id) resetForm();
      render();
    });
  });
}

function resetForm() {
  state.editingId = null;
  elements.formTitle.textContent = 'Thêm giao dịch';
  elements.transactionId.value = '';
  elements.dateInput.value = new Date().toISOString().slice(0, 10);
  elements.typeInput.value = 'income';
  refreshCategoryOptions();
  elements.categoryInput.value = categoryMap.income[0];
  elements.amountInput.value = '';
  elements.noteInput.value = '';
  elements.cancelEditBtn.classList.add('hidden');
}

function formatCurrency(value) {
  return new Intl.NumberFormat('vi-VN', {
    style: 'currency',
    currency: 'VND',
    maximumFractionDigits: 0
  }).format(Number(value || 0));
}

function formatDate(dateString) {
  const date = new Date(dateString);
  return new Intl.DateTimeFormat('vi-VN', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  }).format(date);
}
