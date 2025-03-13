<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Society Maintenance Payment Status</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            text-align: center;
            background-color: #f1f3e5;
        }
        #logo {
            width: 100%;
            max-width: 800px;
            display: block;
            margin: auto;
        }
        table {
            width: 90%;
            margin: auto;
            border-collapse: collapse;
            display: none;
            border: 2px solid black;
        }
        th, td {
            border: 1px solid black;
            padding: 8px;
            text-align: center;
        }
        th {
            background-color: #f4f4f4;
        }
        .paid-row {
            background-color: #d4edda;
            color: #155724;
        }
        .pending-row {
            background-color: #f8d7da !important;
            color: #721c24;
            font-weight: bold;
        }
        .container {
            margin-bottom: 20px;
        }
        input, select, button {
            padding: 8px;
            font-size: 16px;
            margin: 5px;
        }
        #qrcode {
            width: 200px;
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <img id="logo" src="https://raw.githubusercontent.com/Nimeshpatel90/society-maintenance/main/Logo.jpeg" alt="Harikrupa Society Logo">

    <h2>Society Maintenance Payment Status</h2>

    <div class="container">
        <label for="searchHouse">Search by House No.:</label>
        <input type="text" id="searchHouse" placeholder="Enter House No. (e.g., B/07)">

        <label for="statusFilter">Filter by Status:</label>
        <select id="statusFilter" onchange="searchData()">
            <option value="all">All</option>
            <option value="paid">Paid</option>
            <option value="pending">Pending</option>
        </select>

        <button onclick="searchData()">Search</button>
    </div>

    <table id="paymentTable">
        <thead>
            <tr id="tableHeader"></tr>
        </thead>
        <tbody></tbody>
    </table>

    <p id="noResults" style="display:none; color: red; font-weight: bold;">No records found!</p>

    <img id="qrcode" src="qrcode.jpeg" alt="QR Code for Payment">
    <p style="text-align: center; font-weight: bold; color: #333; margin-top: 10px;">
        Please make payment using this QR Code and share payment details to this mobile: <b>9723151500</b>
    </p>

    <script>
        const sheetUrl = "https://docs.google.com/spreadsheets/d/e/2PACX-1vTuo32w3JKptA5vRALRwDIpTK8f7e7clNfXc5VC7RpLDa_nd2e2rVfyqpZ8wEbg9Zp6ESAydUw35DIO/pub?gid=484381785&single=true&output=csv";
        let paymentData = [];
        let headers = [];

        async function fetchData() {
            try {
                console.log("🔄 Fetching latest data...");
                const response = await fetch(sheetUrl + "&t=" + new Date().getTime());
                if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);

                const data = await response.text();
                const rows = data.split("\n").map(row => row.split(","));

                if (rows.length < 2) {
                    console.error("❌ No data found in the sheet!");
                    return;
                }

                headers = rows[0].map(h => h.trim());
                paymentData = rows.slice(1).map(row => {
                    let obj = {};
                    headers.forEach((header, index) => {
                        obj[header] = row[index] ? row[index].trim() : "-";
                    });
                    return obj;
                });

                updateTableHeaders();
            } catch (error) {
                console.error("❌ Error fetching data:", error);
            }
        }

        function updateTableHeaders() {
            let headerRow = document.getElementById("tableHeader");
            headerRow.innerHTML = "";

            headers.forEach(header => {
                let th = document.createElement("th");
                th.textContent = header;
                headerRow.appendChild(th);
            });
        }

        function formatCurrency(amount) {
            return new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" }).format(amount);
        }

        function formatDate(dateString) {
            let date = new Date(dateString);
            if (isNaN(date)) return dateString;
            return date.toLocaleDateString("en-GB");
        }

        function searchData() {
            let searchInput = document.getElementById("searchHouse").value.trim().toUpperCase();
            let statusFilter = document.getElementById("statusFilter").value;
            let tableBody = document.querySelector("#paymentTable tbody");
            let table = document.getElementById("paymentTable");
            let noResults = document.getElementById("noResults");

            tableBody.innerHTML = "";

            let filteredData = paymentData.filter(entry => {
                let matchesHouse = !searchInput || entry["House No."].toUpperCase().includes(searchInput);
                let matchesStatus = statusFilter === "all" ||
                    (statusFilter === "paid" && entry["Payment Status"].toLowerCase().includes("paid")) ||
                    (statusFilter === "pending" && entry["Payment Status"].toLowerCase().includes("pending"));

                return matchesHouse && matchesStatus;
            });

            if (filteredData.length === 0) {
                table.style.display = "none";
                noResults.style.display = "block";
                return;
            }

            noResults.style.display = "none";
            table.style.display = "table";

            filteredData.forEach(entry => {
                let tr = document.createElement("tr");

                if (entry["Payment Status"].toLowerCase().includes("pending")) {
                    tr.classList.add("pending-row");
                } else {
                    tr.classList.add("paid-row");
                }

                headers.forEach(header => {
                    let td = document.createElement("td");

                    if (header.toLowerCase().includes("amount")) {
                        td.textContent = formatCurrency(entry[header]);
                    } else if (header.toLowerCase().includes("date")) {
                        td.textContent = formatDate(entry[header]);
                    } else if (header.toLowerCase().includes("receipt")) {
                        if (entry[header] && entry[header] !== "-") {
                            let link = document.createElement("a");
                            link.href = entry[header];
                            link.target = "_blank";
                            link.textContent = "View Receipt";
                            link.style.color = "blue";
                            link.style.textDecoration = "underline";
                            td.appendChild(link);
                        } else {
                            td.textContent = "-";
                        }
                    } else {
                        td.textContent = entry[header] || "-";
                    }

                    tr.appendChild(td);
                });
                tableBody.appendChild(tr);
            });
        }

        fetchData();
        setInterval(fetchData, 60000);
    </script>
</body>
</html>
