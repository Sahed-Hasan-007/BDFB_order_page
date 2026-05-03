<script setup lang="ts">
import { ref, computed, watch } from "vue";
import Icon from "../components/Icon.vue";
import CommonTutorialModal from "../components/CommonTutorialModal.vue";

type OrderStatus =
  | "Pending"
  | "Followup"
  | "Confirmed"
  | "Canceled"
  | "Ready To Ship"
  | "Shipped"
  | "Hold-by-courier"
  | "Delivered"
  | "Payment-received"
  | "Returned"
  | "Unresolved";

const statuses: (OrderStatus | "All")[] = [
  "Pending", "Followup", "Confirmed", "Canceled", "Ready To Ship",
  "Shipped", "Hold-by-courier", "Delivered", "Payment-received", "Returned", "Unresolved", "All"
];

const periodOptions = [
  { label: "Today", value: "today" },
  { label: "Yesterday", value: "yesterday" },
  { label: "This week", value: "this_week" },
  { label: "This month", value: "this_month" },
  { label: "This year", value: "this_year" },
];

const tutorialOpen = ref(false)
const period = ref("today");
const activeStatus = ref<OrderStatus | "All">("Pending");
const search = ref("");
const page = ref(1);
const pageSize = ref(10);

const statusClass: Record<OrderStatus, string> = {
  "Pending": "bg-amber-50 text-amber-800 dark:bg-amber-500/10 dark:text-amber-400",
  "Followup": "bg-blue-50 text-blue-800 dark:bg-blue-500/10 dark:text-blue-400",
  "Confirmed": "bg-green-50 text-green-800 dark:bg-green-500/10 dark:text-green-400",
  "Canceled": "bg-red-50 text-red-800 dark:bg-red-500/10 dark:text-red-400",
  "Ready To Ship": "bg-indigo-50 text-indigo-800 dark:bg-indigo-500/10 dark:text-indigo-400",
  "Shipped": "bg-violet-50 text-violet-800 dark:bg-violet-500/10 dark:text-violet-400",
  "Hold-by-courier": "bg-primary-50 text-primary-800 dark:bg-primary-500/10 dark:text-primary-400",
  "Delivered": "bg-emerald-50 text-emerald-800 dark:bg-emerald-500/10 dark:text-emerald-400",
  "Payment-received": "bg-teal-50 text-teal-800 dark:bg-teal-500/10 dark:text-teal-400",
  "Returned": "bg-yellow-50 text-yellow-800 dark:bg-yellow-500/10 dark:text-yellow-400",
  "Unresolved": "bg-rose-50 text-rose-800 dark:bg-rose-500/10 dark:text-rose-400",
};

const sourceClass: Record<string, string> = {
  "Facebook": "bg-blue-50 text-blue-800 dark:bg-blue-500/10 dark:text-blue-400",
  "Website": "bg-green-50 text-green-800 dark:bg-green-500/10 dark:text-green-400",
  "WhatsApp": "bg-emerald-50 text-emerald-800 dark:bg-emerald-500/10 dark:text-emerald-400",
  "Walk-in": "bg-purple-50 text-purple-800 dark:bg-purple-500/10 dark:text-purple-400",
  "Phone": "bg-primary-50 text-primary-800 dark:bg-primary-500/10 dark:text-primary-400",
};

interface Order {
  id: string;
  date: string;
  deliveryType: string;
  source: string;
  customerName: string;
  customerPhone: string;
  customerAddress: string;
  products: { name: string; qty: number }[];
  subtotal: number;
  deliveryFee: number;
  discount: number;
  paid: number;
  courierName: string;
  courierTracking: string;
  status: OrderStatus;
  agentName: string;
}

const orders = ref<Order[]>([
  {
    id: "#ORD-3001", date: "13 Apr, 09:24 AM", deliveryType: "Home delivery", source: "Facebook",
    customerName: "Karim Hassan", customerPhone: "01712-345678", customerAddress: "Mirpur-10, Dhaka",
    products: [{ name: "Cotton Polo Shirt", qty: 2 }, { name: "Slim Fit Jeans", qty: 1 }],
    subtotal: 2400, deliveryFee: 80, discount: 100, paid: 2380, courierName: "Pathao", courierTracking: "TRK100001", status: "Confirmed", agentName: "Rahim Uddin",
  },
  {
    id: "#ORD-3002", date: "13 Apr, 09:11 AM", deliveryType: "Express", source: "Website",
    customerName: "Rima Akter", customerPhone: "01812-234567", customerAddress: "Gulshan-2, Dhaka",
    products: [{ name: "Wireless Earbuds", qty: 1 }],
    subtotal: 1800, deliveryFee: 120, discount: 0, paid: 0, courierName: "Steadfast", courierTracking: "TRK100002", status: "Pending", agentName: "Farhan Ali",
  },
  {
    id: "#ORD-3003", date: "13 Apr, 08:58 AM", deliveryType: "Standard", source: "WhatsApp",
    customerName: "Sajib Mondal", customerPhone: "01912-876543", customerAddress: "Uttara Sector-7",
    products: [{ name: "Running Shoes", qty: 1 }, { name: "Sports Socks", qty: 3 }],
    subtotal: 3500, deliveryFee: 100, discount: 200, paid: 3400, courierName: "Redx", courierTracking: "TRK100003", status: "Shipped", agentName: "Farhan Ali",
  },
  {
    id: "#ORD-3004", date: "13 Apr, 08:43 AM", deliveryType: "Home delivery", source: "Phone",
    customerName: "Meherun Nessa", customerPhone: "01611-456789", customerAddress: "Mohammadpur, Dhaka",
    products: [{ name: "Leather Wallet", qty: 1 }, { name: "Belt", qty: 1 }],
    subtotal: 950, deliveryFee: 60, discount: 0, paid: 0, courierName: "eCourier", courierTracking: "TRK100004", status: "Canceled", agentName: "Nadia Islam",
  },
  {
    id: "#ORD-3005", date: "13 Apr, 08:32 AM", deliveryType: "Home delivery", source: "Facebook",
    customerName: "Jahangir Alam", customerPhone: "01511-654321", customerAddress: "Dhanmondi-27",
    products: [{ name: "Smart Watch", qty: 1 }],
    subtotal: 5500, deliveryFee: 0, discount: 500, paid: 5000, courierName: "Pathao", courierTracking: "TRK100005", status: "Delivered", agentName: "Rahim Uddin",
  },
  {
    id: "#ORD-3006", date: "13 Apr, 08:17 AM", deliveryType: "Store pickup", source: "Walk-in",
    customerName: "Taslima Begum", customerPhone: "01711-123456", customerAddress: "Banani, Dhaka",
    products: [{ name: "Face Cream", qty: 2 }, { name: "Serum", qty: 1 }, { name: "Toner", qty: 1 }],
    subtotal: 2200, deliveryFee: 0, discount: 150, paid: 1000, courierName: "Sundarban", courierTracking: "TRK100006", status: "Followup", agentName: "Tasnim Khanam",
  },
  {
    id: "#ORD-3007", date: "13 Apr, 08:05 AM", deliveryType: "Express", source: "Website",
    customerName: "Rafiqul Islam", customerPhone: "01811-345678", customerAddress: "Wari, Dhaka",
    products: [{ name: "Formal Shirt", qty: 1 }, { name: "Trouser", qty: 1 }],
    subtotal: 3200, deliveryFee: 80, discount: 0, paid: 3280, courierName: "Redx", courierTracking: "TRK100007", status: "Ready To Ship", agentName: "Farhan Ali",
  },
  {
    id: "#ORD-3008", date: "13 Apr, 07:54 AM", deliveryType: "Standard", source: "Facebook",
    customerName: "Shirin Sultana", customerPhone: "01711-987654", customerAddress: "Narayanganj",
    products: [{ name: "Kurti Set", qty: 2 }],
    subtotal: 1400, deliveryFee: 100, discount: 0, paid: 1500, courierName: "Pathao", courierTracking: "TRK100008", status: "Returned", agentName: "Sumaiya Begum",
  },
  {
    id: "#ORD-3009", date: "13 Apr, 07:41 AM", deliveryType: "Home delivery", source: "WhatsApp",
    customerName: "Monir Hossain", customerPhone: "01612-543210", customerAddress: "Gazipur Sadar",
    products: [{ name: "Laptop Bag", qty: 1 }],
    subtotal: 1900, deliveryFee: 120, discount: 100, paid: 1920, courierName: "Steadfast", courierTracking: "TRK100009", status: "Payment-received", agentName: "Nadia Islam",
  },
  {
    id: "#ORD-3010", date: "13 Apr, 07:28 AM", deliveryType: "Home delivery", source: "Phone",
    customerName: "Nasrin Khatun", customerPhone: "01912-112233", customerAddress: "Tongi, Gazipur",
    products: [{ name: "Baby Onesie", qty: 3 }, { name: "Baby Cap", qty: 2 }],
    subtotal: 1600, deliveryFee: 80, discount: 0, paid: 0, courierName: "eCourier", courierTracking: "TRK100010", status: "Unresolved", agentName: "Tasnim Khanam",
  },
  {
    id: "#ORD-3011", date: "13 Apr, 07:10 AM", deliveryType: "Express", source: "Website",
    customerName: "Arif Hossain", customerPhone: "01712-998877", customerAddress: "Dhanmondi-32, Dhaka",
    products: [{ name: "Sneakers", qty: 1 }],
    subtotal: 4200, deliveryFee: 0, discount: 200, paid: 4000, courierName: "Pathao", courierTracking: "TRK100011", status: "Hold-by-courier", agentName: "Rahim Uddin",
  },
  {
    id: "#ORD-3012", date: "12 Apr, 06:50 PM", deliveryType: "Standard", source: "Facebook",
    customerName: "Nusrat Jahan", customerPhone: "01611-774455", customerAddress: "Mirpur-1, Dhaka",
    products: [{ name: "Hijab (Chiffon)", qty: 4 }],
    subtotal: 1200, deliveryFee: 60, discount: 60, paid: 1200, courierName: "Sundarban", courierTracking: "TRK100012", status: "Delivered", agentName: "Farhan Ali",
  },
  {
    id: "#ORD-3013", date: "12 Apr, 05:30 PM", deliveryType: "Home delivery", source: "WhatsApp",
    customerName: "Saiful Islam", customerPhone: "01811-336699", customerAddress: "Rampura, Dhaka",
    products: [{ name: "Gym Gloves", qty: 1 }, { name: "Protein Shaker", qty: 2 }],
    subtotal: 950, deliveryFee: 80, discount: 0, paid: 500, courierName: "Redx", courierTracking: "TRK100013", status: "Pending", agentName: "Sumaiya Begum",
  },
  {
    id: "#ORD-3014", date: "12 Apr, 04:15 PM", deliveryType: "Store pickup", source: "Walk-in",
    customerName: "Fatima Khan", customerPhone: "01912-556677", customerAddress: "Khilgaon, Dhaka",
    products: [{ name: "Sunscreen SPF 50", qty: 2 }, { name: "Lip Balm", qty: 3 }],
    subtotal: 1100, deliveryFee: 0, discount: 100, paid: 1000, courierName: "eCourier", courierTracking: "TRK100014", status: "Confirmed", agentName: "Nadia Islam",
  },
  {
    id: "#ORD-3015", date: "12 Apr, 03:00 PM", deliveryType: "Express", source: "Facebook",
    customerName: "Rahul Ahmed", customerPhone: "01712-223344", customerAddress: "Badda, Dhaka",
    products: [{ name: "Formal Blazer", qty: 1 }],
    subtotal: 6500, deliveryFee: 0, discount: 500, paid: 6000, courierName: "Steadfast", courierTracking: "TRK100015", status: "Shipped", agentName: "Tasnim Khanam",
  },
  {
    id: "#ORD-3016", date: "12 Apr, 01:45 PM", deliveryType: "Home delivery", source: "Phone",
    customerName: "Dilruba Khanam", customerPhone: "01611-998800", customerAddress: "Demra, Dhaka",
    products: [{ name: "Cotton Saree", qty: 1 }],
    subtotal: 2800, deliveryFee: 100, discount: 0, paid: 0, courierName: "Pathao", courierTracking: "TRK100016", status: "Followup", agentName: "Rahim Uddin",
  },
  {
    id: "#ORD-3017", date: "12 Apr, 12:30 PM", deliveryType: "Standard", source: "Website",
    customerName: "Mosharraf Hossain", customerPhone: "01812-667788", customerAddress: "Motijheel, Dhaka",
    products: [{ name: "Leather Shoes", qty: 1 }, { name: "Shoe Polish", qty: 1 }],
    subtotal: 3800, deliveryFee: 80, discount: 200, paid: 3680, courierName: "Sundarban", courierTracking: "TRK100017", status: "Ready To Ship", agentName: "Farhan Ali",
  },
  {
    id: "#ORD-3018", date: "12 Apr, 11:10 AM", deliveryType: "Home delivery", source: "WhatsApp",
    customerName: "Sumaiya Islam", customerPhone: "01712-443322", customerAddress: "Shyamoli, Dhaka",
    products: [{ name: "Kids T-Shirt", qty: 3 }, { name: "Kids Shorts", qty: 2 }],
    subtotal: 1500, deliveryFee: 60, discount: 0, paid: 1560, courierName: "Redx", courierTracking: "TRK100018", status: "Payment-received", agentName: "Sumaiya Begum",
  },
  {
    id: "#ORD-3019", date: "12 Apr, 10:00 AM", deliveryType: "Express", source: "Facebook",
    customerName: "Tanvir Ahmed", customerPhone: "01911-554433", customerAddress: "Lalmatia, Dhaka",
    products: [{ name: "Wireless Mouse", qty: 1 }, { name: "Keyboard", qty: 1 }],
    subtotal: 3200, deliveryFee: 0, discount: 300, paid: 0, courierName: "eCourier", courierTracking: "TRK100019", status: "Unresolved", agentName: "Nadia Islam",
  },
  {
    id: "#ORD-3020", date: "12 Apr, 09:00 AM", deliveryType: "Home delivery", source: "Phone",
    customerName: "Khaleda Akter", customerPhone: "01611-221133", customerAddress: "Jatrabari, Dhaka",
    products: [{ name: "Pressure Cooker", qty: 1 }],
    subtotal: 2600, deliveryFee: 150, discount: 0, paid: 1500, courierName: "Steadfast", courierTracking: "TRK100020", status: "Hold-by-courier", agentName: "Tasnim Khanam",
  },
]);

const filtered = computed(() =>
  orders.value.filter((o) => {
    const sm = activeStatus.value === "All" || o.status === activeStatus.value;
    const q = search.value.toLowerCase();
    const qm = !q || o.id.toLowerCase().includes(q) || o.products.some((p) => p.name.toLowerCase().includes(q));
    return sm && qm;
  })
);

const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / pageSize.value)));
const paginated = computed(() => filtered.value.slice((page.value - 1) * pageSize.value, page.value * pageSize.value));

watch([activeStatus, search, pageSize], () => { page.value = 1; });

const grandTotal = (o: Order) => o.subtotal + o.deliveryFee - o.discount;
const due = (o: Order) => Math.max(0, grandTotal(o) - o.paid);

function exportOrders() {
  const blob = new Blob([JSON.stringify(orders.value, null, 2)], { type: "application/json" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "orders.json";
  a.click();
}
</script>

<template>
  <div class="p-6 font-sans text-gray-900 dark:text-zinc-100 antialiased bg-[#0a0a0b] min-h-screen">

    <!-- Page header -->
    <div class="flex items-start justify-between flex-wrap gap-4 mb-6">
      <div>
        <nav class="flex items-center gap-1 mb-1">
          <span
            class="text-xs font-medium text-zinc-500 cursor-pointer hover:text-zinc-900 dark:hover:text-zinc-200">Orders</span>
          <Icon name="lucide:chevron-right" class="w-3.5 h-3.5 text-zinc-400" />
          <span class="text-xs font-medium text-zinc-400">List</span>
        </nav>
        <h1 class="text-2xl font-bold tracking-tight text-zinc-900 dark:text-white m-0">Orders</h1>
      </div>
      <div class="flex items-center gap-2 flex-wrap">
        <button
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#1a1a1c] dark:hover:bg-[#2a2a2c] cursor-pointer">
          Show Duplicate Orders
        </button>
        <button
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#1a1a1c] dark:hover:bg-[#2a2a2c] cursor-pointer">
          Me Mode
        </button>
        <button
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#1a1a1c] dark:hover:bg-[#2a2a2c] cursor-pointer">
          Unassigned Orders
        </button>
        <button
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#1a1a1c] dark:hover:bg-[#2a2a2c] cursor-pointer">
          Sort By Update
        </button>
        <button @click="exportOrders"
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#c65f00] dark:hover:bg-[#8d4705] cursor-pointer">
          <Icon name="lucide:download" class="w-4 h-4 shrink-0 text-primary-500" />
          Export
        </button>
        <button
          class="flex items-center gap-2 rounded-lg border border-zinc-200 px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:text-zinc-200 dark:hover:border-primary-500 bg-white dark:bg-[#c65f00] dark:hover:bg-[#8d4705] cursor-pointer">
          New order
        </button>
        <button @click="tutorialOpen = true"
          class="flex items-center gap-2 rounded-lg border border-zinc-200 bg-white px-3 py-2 text-sm font-semibold text-zinc-700 transition-all hover:border-primary-400 dark:border-zinc-700 dark:bg-[#009689] dark:hover:bg-[#156b64] dark:text-zinc-200 dark:hover:border-primary-500 cursor-pointer">
          <Icon name="lucide:play-circle" class="w-4 h-4 shrink-0 text-primary-500" />
          Tutorial
        </button>
      </div>
    </div>

    <!-- Tutorial Modal -->
    <CommonTutorialModal v-if="tutorialOpen" video-url="https://youtu.be/GNM5ViJEEK0" title="Orders — Tutorial"
      @close="tutorialOpen = false" />

    <!-- Period selector -->
    <div
      class="flex items-center mb-5 flex-wrap p-5 bg-white dark:bg-[#18181c] border border-zinc-200 dark:border-zinc-700 rounded-xl">
      <div
        class="text-sm h-9 text-zinc-700 dark:text-zinc-200 rounded-l-lg border border-zinc-200 dark:border-zinc-700 bg-white dark:bg-[#27272a] flex items-center px-2">
        Filter by</div>
      <div class="relative">
        <select v-model="period"
          class="pl-2 pr-4 h-9 w-[400px] rounded-r-lg border border-zinc-200 dark:border-zinc-700 bg-white dark:bg-[#27272a] text-sm font-semibold text-zinc-700 dark:text-zinc-200 outline-none focus:border-primary-400 cursor-pointer">
          <option v-for="opt in periodOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
        </select>
      </div>
      <span class="text-xs font-medium text-zinc-400 dark:text-zinc-500 ml-3">{{ filtered.length }} order{{
        filtered.length !== 1 ? 's' : '' }} found</span>
    </div>

    <!-- Status tabs -->
    <div
      class="bg-white w-fit mx-auto flex items-center justify-center dark:bg-[#18181c] mb-5 rounded-2xl border border-zinc-200 dark:border-zinc-700 overflow-hidden shadow-sm">
      <div class="flex items-center flex-wrap gap-1.5 px-4 py-3">
        <button v-for="s in statuses" :key="s" @click="activeStatus = s; page = 1" :class="[
          'px-3 py-1.5 rounded-lg text-[13px] font-semibold transition-all whitespace-nowrap',
          activeStatus === s
            ? 'bg-primary-50 text-primary-700 dark:bg-orange-200/10 dark:text-[#de883e]'
            : 'text-zinc-500 border-transparent hover:bg-zinc-100 hover:text-zinc-700 dark:text-zinc-400 dark:hover:bg-zinc-800 dark:hover:text-zinc-200'
        ]">
          {{ s }}
        </button>
      </div>
    </div>

    <!-- Main card -->
    <div
      class="bg-white dark:bg-zinc-900 rounded-2xl border border-zinc-200 dark:border-zinc-700 overflow-hidden shadow-sm">

      <!-- Toolbar: search -->
      <div
        class="flex items-center justify-end px-4 py-3 border-b border-zinc-200 dark:border-zinc-700  gap-3 flex-wrap">
        <div class="flex items-center gap-2">
          <div class="relative flex items-center">
            <Icon name="lucide:search" class="absolute left-2.5 w-4 h-4 text-zinc-400 pointer-events-none" />
            <input v-model="search" type="search" placeholder="Search"
              class="h-9 pl-8 pr-3 w-52 border border-zinc-200 dark:border-zinc-700 rounded-lg text-[13px] text-zinc-900 dark:text-zinc-100 bg-white dark:bg-zinc-800 outline-none placeholder-zinc-400 dark:placeholder-zinc-600 focus:border-orange-400 focus:ring-2 focus:ring-orange-400/10 transition-all" />
          </div>
          <div
            class="relative flex items-center justify-center w-9 h-9 rounded-lg bg-white dark:bg-zinc-900 cursor-pointer hover:bg-zinc-50 dark:hover:bg-zinc-700 transition-colors">
            <Icon name="material-symbols:filter-alt" class="w-6 h-6 text-zinc-600 dark:text-zinc-500" />
            <span
              class="absolute -top-1 -right-1 flex items-center justify-center w-4 h-4 text-[10px] font-bold text-[#f9c401] bg-yellow-400/20 rounded-full ring-2 ring-white dark:ring-zinc-900 border border-[#77610e]">
              2
            </span>
          </div>
        </div>
      </div>

      <!-- Table -->
      <div class="overflow-x-auto">
        <table class="w-full border-collapse text-[12.5px]" style="min-width:1100px">
          <!-- <thead>
            <tr>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-44">Order info</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-44">Customer</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-52">Products</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-48">Financials</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-40">Courier</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-left text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-36">Status</th>
                <th class="bg-primary-50 dark:bg-primary-950/10 px-3.5 py-2.5 text-center text-[11px] font-bold text-zinc-400 uppercase tracking-widest border-b border-primary-200 dark:border-primary-900/20 whitespace-nowrap w-28">Actions</th>
            </tr>
          </thead> -->
          <tbody>

            <!-- Empty -->
            <tr v-if="paginated.length === 0">
              <td colspan="7" class="text-center p-12">
                <div class="flex flex-col items-center gap-2 text-zinc-400">
                  <Icon name="lucide:inbox" class="w-9 h-9" />
                  <p class="text-sm m-0">No orders found</p>
                </div>
              </td>
            </tr>

            <!-- Rows -->
            <tr v-for="order in paginated" :key="order.id"
              class="hover:bg-primary-50/50 dark:hover:bg-primary-950/5 transition-colors">
              <!-- Order info -->
              <td class="px-4 py-3 border-b border-zinc-200 dark:border-zinc-700 align-center">

                <!-- Order ID -->
                <div class="text-[#00b8da] font-semibold text-sm mb-1">
                  {{ order.id }}
                </div>

                <!-- Date -->
                <div class="text-sm text-zinc-400 dark:text-zinc-300 mb-2">
                  {{ order.date }}
                </div>

                <!-- Delivery Type Badge -->
                <div class="flex flex-col gap-1">
                  <span class="w-fit px-2.5 py-0.5 text-[13px] font-medium rounded-md bg-indigo-700/10 text-indigo-200 border border-indigo-500/20">
                    {{ order.source }}
                  </span>
                  <span class="w-fit px-2.5 py-0.5 text-[13px] font-medium rounded-md border bg-orange-500/10 text-orange-300 border-orange-500/20">
                    {{ order.deliveryType }}
                  </span>
                </div>

              </td>

              <!-- Customer -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center text-[14px]">
                <div class="font-semibold text-zinc-900 dark:text-zinc-100 mb-0.5">{{ order.customerName
                  }}</div>
                <div class="text-zinc-500 dark:text-zinc-100 mb-0.5">{{ order.customerPhone
                  }}</div>
                <div class="text-zinc-400 dark:text-zinc-100">{{ order.customerAddress }}
                </div>
              </td>
              <!-- Products -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center">
                <div v-for="p in order.products" :key="p.name" class="flex items-center gap-1 mb-1 last:mb-0">
                  <span class="text-zinc-700 dark:text-[#0a91b2] text-[15px]">{{ p.name }}</span>
                  <span class="text-[14px] font-bold text-primary-400 ml-0.5">×{{ p.qty }}</span>
                </div>
              </td>
              <!-- Financials -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center">
                <div class="grid grid-cols-2 gap-y-0.5 text-[12px]">
                  <span class="text-zinc-400">Subtotal</span>
                  <span class="text-zinc-700 dark:text-zinc-300 font-medium">৳{{
                    order.subtotal.toLocaleString() }}</span>
                  <span class="text-zinc-400">Delivery</span>
                  <span class="text-zinc-600 dark:text-zinc-400">৳{{ order.deliveryFee }}</span>
                  <span class="text-zinc-400">Discount</span>
                  <span class="text-zinc-600 dark:text-zinc-400">-৳{{ order.discount }}</span>
                  <span class="text-zinc-400">Paid</span>
                  <span class="font-semibold text-green-600 dark:text-green-400">৳{{
                    order.paid.toLocaleString() }}</span>
                  <span class="text-zinc-400">Due</span>
                  <span class="font-semibold"
                    :class="due(order) > 0 ? 'text-red-600 dark:text-red-400' : 'text-green-600 dark:text-green-400'">৳{{
                      due(order).toLocaleString() }}</span>
                  <span
                    class="text-zinc-500 font-semibold pt-1">Grand</span>
                  <span
                    class="font-bold text-primary-500 pt-1">৳{{
                      grandTotal(order).toLocaleString() }}</span>
                </div>
              </td>

              <!-- Courier -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center">
                <div class="flex items-center text-[14px] bg-white dark:bg-[#c65f00] gap-1.5 p-2 rounded-lg font-semibold text-zinc-800 dark:text-zinc-200 mb-0.5 w-fit">
                  <Icon name="material-symbols:delivery-truck-speed" class="w-5 h-5 text-zinc-800 dark:text-zinc-200 shrink-0" />
                  {{ order.courierName }}
                </div>
              </td>

              <!-- Status -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center">
                <div class="flex items-center text-[14px] bg-white dark:bg-[#155cfc] gap-1.5 p-2 rounded-lg font-semibold text-zinc-800 dark:text-zinc-200 mb-0.5 w-fit">
                  <Icon name="material-symbols:android-chat" class="w-5 h-5 text-zinc-800 dark:text-zinc-200 shrink-0" />
                  {{ order.status }}
                </div>
              </td>

              <!-- Agent -->
              <!-- Agent -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700  align-center">
                <div class="flex items-center text-[14px] bg-white dark:bg-[#009689] gap-1.5 p-2 rounded-lg font-semibold text-zinc-800 dark:text-zinc-200 mb-0.5 w-fit">
                  <Icon name="lucide:user" class="w-4 h-4 text-zinc-800 dark:text-zinc-200 shrink-0" />
                  {{ order.agentName.slice(0, 2) }}
                </div>
              </td>

              <!-- Actions -->
              <td class="px-3.5 py-3 border-b border-zinc-200 dark:border-zinc-700 align-center">
                <div class="flex items-center justify-center gap-3 flex-wrap">
                  <!-- Print -->
                  <div class="relative group">
                    <button class="inline-flex items-center justify-center w-6 h-6 rounded-lg text-orange-400 cursor-pointer hover:text-zinc-700 dark:hover:text-[#c65f00] transition-all">
                      <Icon name="heroicons:printer-solid" class="w-5 h-5" />
                    </button>
                    <div class="pointer-events-none absolute bottom-full left-1/2 -translate-x-1/2 mb-1.5 px-2 py-1 rounded-md bg-zinc-900 dark:bg-zinc-700 text-white text-[11px] font-medium whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-0 z-50">
                      Print Invoice
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-zinc-900 dark:border-t-zinc-700"></div>
                    </div>
                  </div>
                  <!-- Received Paid Amount -->
                  <div class="relative group">
                    <button class="inline-flex items-center justify-center w-6 h-6 rounded-lg text-orange-400 cursor-pointer hover:text-emerald-600 dark:hover:text-[#c65f00] transition-all">
                      <Icon name="heroicons:document-currency-bangladeshi-solid" class="w-4 h-4" />
                    </button>
                    <div class="pointer-events-none absolute bottom-full left-1/2 -translate-x-1/2 mb-1.5 px-2 py-1 rounded-md bg-zinc-900 dark:bg-zinc-700 text-white text-[11px] font-medium whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-0 z-50">
                      Received Paid Amount
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-zinc-900 dark:border-t-zinc-700"></div>
                    </div>
                  </div>
                  <!-- Copy -->
                  <div class="relative group">
                    <button class="inline-flex items-center justify-center w-6 h-6 rounded-lg text-orange-400 cursor-pointer hover:text-zinc-700 dark:hover:text-[#c65f00] transition-all">
                      <Icon name="pixel:copy-solid" class="w-4 h-4" />
                    </button>
                    <div class="pointer-events-none absolute bottom-full left-1/2 -translate-x-1/2 mb-1.5 px-2 py-1 rounded-md bg-zinc-900 dark:bg-zinc-700 text-white text-[11px] font-medium whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-0 z-50">
                      Copy Order
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-zinc-900 dark:border-t-zinc-700"></div>
                    </div>
                  </div>
                  <!-- View -->
                  <div class="relative group">
                    <button class="inline-flex items-center justify-center w-6 h-6 rounded-lg text-green-400 cursor-pointer hover:text-blue-600 dark:hover:text-[#c65f00] transition-all">
                      <Icon name="mdi:eye" class="w-4 h-4" />
                    </button>
                    <div class="pointer-events-none absolute bottom-full left-1/2 -translate-x-1/2 mb-1.5 px-2 py-1 rounded-md bg-zinc-900 dark:bg-zinc-700 text-white text-[11px] font-medium whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-0 z-50">
                      View Details
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-zinc-900 dark:border-t-zinc-700"></div>
                    </div>
                  </div>
                  <!-- Edit -->
                  <div class="relative group">
                    <button class="inline-flex items-center justify-center w-6 h-6 rounded-lg text-orange-400 cursor-pointer hover:text-green-600 dark:hover:text-[#c65f00] transition-all">
                      <Icon name="material-symbols:edit-square-outline-rounded" class="w-4 h-4" />
                    </button>
                    <div class="pointer-events-none absolute bottom-full left-1/2 -translate-x-1/2 mb-1.5 px-2 py-1 rounded-md bg-zinc-900 dark:bg-zinc-700 text-white text-[11px] font-medium whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity duration-0 z-50">
                      Edit Order
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-zinc-900 dark:border-t-zinc-700"></div>
                    </div>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination -->
      <div
        class="flex flex-col md:flex-row items-center justify-between px-4 py-3 border-primary-100 dark:border-primary-900/15 gap-2">
        <span class="text-xs font-medium text-zinc-400 dark:text-zinc-500 tracking-wide">
          Showing {{ paginated.length ? (page - 1) * pageSize + 1 : 0 }}–{{ Math.min(page * pageSize,
            filtered.length) }} of {{ filtered.length }} results
        </span>
        <div class="flex items-center gap-4">
          <div class="flex items-center gap-2">
            <label class="text-xs font-medium text-zinc-500 dark:text-zinc-400">Rows per page:</label>
            <select v-model.number="pageSize"
              class="border border-zinc-200 dark:border-zinc-700 rounded-lg px-2 py-1 text-xs font-semibold text-zinc-700 dark:text-zinc-300 bg-white dark:bg-zinc-800 outline-none focus:border-primary-400">
              <option v-for="s in [10, 25, 50]" :key="s" :value="s">{{ s }}</option>
            </select>
          </div>
          <div class="flex items-center gap-1.5">
            <button :disabled="page <= 1" @click="page--"
              class="px-2.5 py-1.5 border border-zinc-200 dark:border-zinc-700 rounded-lg text-xs font-semibold bg-white dark:bg-zinc-800 text-zinc-500 dark:text-zinc-400 cursor-pointer hover:bg-zinc-50 dark:hover:bg-zinc-700 hover:border-zinc-400 disabled:opacity-35 disabled:cursor-not-allowed transition-all">
              Previous
            </button>
            <span class="text-[13px] font-bold text-zinc-900 dark:text-zinc-100 px-1">{{ page }} / {{
              totalPages }}</span>
            <button :disabled="page >= totalPages" @click="page++"
              class="px-2.5 py-1.5 border border-zinc-200 dark:border-zinc-700 rounded-lg text-xs font-semibold bg-white dark:bg-zinc-800 text-zinc-500 dark:text-zinc-400 cursor-pointer hover:bg-zinc-50 dark:hover:bg-zinc-700 hover:border-zinc-400 disabled:opacity-35 disabled:cursor-not-allowed transition-all">
              Next
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
