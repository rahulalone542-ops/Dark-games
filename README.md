# Finance-DW
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Finance Panel</title>

<style>
body {
    margin: 0;
    font-family: "Segoe UI", Arial, sans-serif;
    background: #eef1f5;
}

/* MENU */
.menu {
    background: linear-gradient(90deg, #1e1e2f, #2d2d4f);
    padding: 15px;
    display: flex;
    justify-content: center;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.menu a {
    color: white;
    text-decoration: none;
    padding: 0 25px;
    font-size: 18px;
    cursor: pointer;
    font-weight: 500;
    position: relative;
    transition: all 0.3s ease;
}

.menu a:not(:last-child) {
    border-right: 2px solid rgba(255,255,255,0.3);
}

.menu a:hover {
    color: #00ffcc;
}

.menu a::after {
    content: '';
    display: block;
    height: 3px;
    width: 0;
    background: #00ffcc;
    transition: width 0.3s;
    position: absolute;
    bottom: -5px;
    left: 0;
}

.menu a:hover::after {
    width: 100%;
}

/* CONTENT */
.content {
    width: 90%;
    margin: 30px auto;
    background: white;
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.1);
    opacity: 0;
    transform: translateY(20px);
    animation: fadeIn 0.5s forwards;
}

@keyframes fadeIn {
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* INPUT AREA */
.input-box {
    margin-bottom: 20px;
}

.input-box input {
    padding: 10px;
    font-size: 16px;
    width: 180px;
    margin-right: 10px;
    border-radius: 6px;
    border: 1px solid #ccc;
}

.input-box button {
    padding: 10px 18px;
    font-size: 16px;
    border-radius: 6px;
    border: none;
    background: #1e1e2f;
    color: white;
    cursor: pointer;
}

.input-box button:hover {
    background: #2d2d4f;
}

/* TABLE */
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    font-size: 15px;
}

th, td {
    padding: 12px;
    border: 1px solid #ddd;
    text-align: center;
}

th {
    background: #1e1e2f;
    color: white;
}

tr:nth-child(even) {
    background: #f7f9fc;
}

.highlight {
    font-weight: bold;
    color: #1e1e2f;
}

.date-cell {
    font-size: 14px;
    color: #555;
}

button.edit-btn {
    background: #00aaff;
}

button.edit-btn:hover {
    background: #0077aa;
}

button.update-btn {
    background: #28a745;
}

button.update-btn:hover {
    background: #1e7e34;
}
</style>
</head>

<body>

<div class="menu">
    <a onclick="showPage('home')">Home</a>
    <a onclick="showPage('deposit')">Deposit</a>
    <a onclick="showPage('withdrawal')">Withdrawal</a>
    <a onclick="showPage('charges')">Charges</a>
</div>

<div class="content" id="content"></div>

<script>
let chargePercent = 3;
let deposits = [];
let withdrawals = [];
let editingIndex = null;
let editingType = null;

/* PAGE SWITCH WITH FADE ANIMATION */
function showPage(page) {
    let content = document.getElementById("content");
    content.style.opacity = 0;
    setTimeout(() => {
        if (page === "home") {
            content.innerHTML = `
                <h2>Dashboard</h2>
                <p class="highlight">Total Deposits: ₹${totalAmount(deposits)}</p>
                <p class="highlight">Total Withdrawals: ₹${totalAmount(withdrawals)}</p>
                <p>Charges Applied: ${chargePercent}%</p>
            `;
        }

        if (page === "deposit") {
            content.innerHTML = `
                <h2>Deposit</h2>

                <div class="input-box">
                    <input type="text" id="depName" placeholder="Name">
                    <input type="number" id="depAmount" placeholder="Amount">
                    <button onclick="addOrUpdate('Deposit')">${editingType === 'Deposit' ? 'Update' : 'Add'}</button>
                </div>

                ${renderTable(deposits, "Deposit")}
            `;
        }

        if (page === "withdrawal") {
            content.innerHTML = `
                <h2>Withdrawal</h2>

                <div class="input-box">
                    <input type="text" id="withName" placeholder="Name">
                    <input type="number" id="withAmount" placeholder="Amount">
                    <button onclick="addOrUpdate('Withdrawal')">${editingType === 'Withdrawal' ? 'Update' : 'Add'}</button>
                </div>

                ${renderTable(withdrawals, "Withdrawal")}
            `;
        }

        if (page === "charges") {
            content.innerHTML = `
                <h2>Charges (%)</h2>

                <input type="number" value="${chargePercent}" 
                    onchange="setCharge(this.value)"> %

                <p>Current Charge: <b>${chargePercent}%</b></p>
            `;
        }

        content.style.opacity = 1;
    }, 200);
}

/* ADD OR UPDATE ENTRY */
function addOrUpdate(type) {
    let nameInput = type === 'Deposit' ? document.getElementById("depName") : document.getElementById("withName");
    let amountInput = type === 'Deposit' ? document.getElementById("depAmount") : document.getElementById("withAmount");
    let name = nameInput.value.trim();
    let amount = Number(amountInput.value);

    if (!name || amount <= 0) return alert("Enter valid Name and Amount");

    if (editingIndex !== null && editingType === type) {
        // Update existing
        let targetArray = type === 'Deposit' ? deposits : withdrawals;
        targetArray[editingIndex] = { ...targetArray[editingIndex], name, amount, date: new Date() };
        editingIndex = null;
        editingType = null;
    } else {
        // Add new
        if (type === 'Deposit') deposits.push({ name, amount, date: new Date() });
        else withdrawals.push({ name, amount, date: new Date() });
    }

    showPage(type.toLowerCase());
}

/* EDIT FUNCTION */
function editEntry(type, index) {
    let targetArray = type === 'Deposit' ? deposits : withdrawals;
    let entry = targetArray[index];
    if (type === 'Deposit') {
        document.getElementById("depName").value = entry.name;
        document.getElementById("depAmount").value = entry.amount;
    } else {
        document.getElementById("withName").value = entry.name;
        document.getElementById("withAmount").value = entry.amount;
    }
    editingIndex = index;
    editingType = type;
    showPage(type.toLowerCase());
}

/* TABLE RENDER */
function renderTable(list, type) {
    if (list.length === 0) return "<p>No records yet</p>";

    let rows = list.map((item, index) => {
        let charge = calculateCharge(item.amount);
        let finalAmt = item.amount - charge;
        let dateStr = item.date.toLocaleDateString();
        return `
            <tr>
                <td>${index + 1}</td>
                <td>${item.name}</td>
                <td>₹${item.amount}</td>
                <td>₹${charge}</td>
                <td>₹${finalAmt}</td>
                <td class="date-cell">${dateStr}</td>
                <td><button class="edit-btn" onclick="editEntry('${type}', ${index})">Edit</button></td>
            </tr>
        `;
    }).join("");

    return `
        <table>
            <tr>
                <th>Sr. No</th>
                <th>Name</th>
                <th>${type} Amount</th>
                <th>Charges (${chargePercent}%)</th>
                <th>Final Amount</th>
                <th>Date</th>
                <th>Edit</th>
            </tr>
            ${rows}
        </table>
    `;
}

/* HELPERS */
function calculateCharge(amount) {
    return Math.round(amount * chargePercent / 100);
}

function totalAmount(list) {
    return list.reduce((sum, item) => sum + item.amount, 0);
}

function setCharge(value) {
    chargePercent = Number(value);
    showPage("charges");
}

/* DEFAULT */
showPage("home");
</script>

</body>
</html>
