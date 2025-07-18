# b7scalc
B7s Tier Calculator
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Revenue Change Calculator</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for Inter font */
        body {
            font-family: "Inter", sans-serif;
            padding-left: 0; /* Removed horizontal padding to allow full width */
            padding-right: 0; /* Removed horizontal padding to allow full width */
        }
        /* Hide number input arrows */
        input[type='number']::-webkit-inner-spin-button,
        input[type='number']::-webkit-outer-spin-button {
            -webkit-appearance: none;
            margin: 0;
        }
        input[type='number'] {
            -moz-appearance: textfield;
        }
        /* Ensure table cells don't wrap too much */
        .min-w-full th, .min-w-full td {
            white-space: nowrap;
        }
        /* Adjust input width for better display in table */
        .variant-input {
            width: 70px; /* Smaller width for number inputs in table */
            padding: 0.25rem 0.5rem; /* Smaller padding */
            font-size: 0.875rem; /* sm:text-sm equivalent */
            text-align: center;
        }
        /* Sticky column for variant name */
        .sticky-col {
            position: sticky;
            left: 0;
            background-color: white;
            z-index: 10;
            box-shadow: 2px 0 5px rgba(0,0,0,0.1); /* Optional shadow for effect */
        }
        /* Adjust sticky column background for header */
        .sticky-col-header {
            background-color: #f9fafb; /* bg-gray-50 */
            z-index: 20; /* Higher z-index for header sticky */
        }
        /* Styling for result tables */
        #results-section table th, #results-section table td {
            padding: 0.5rem 0.75rem;
            text-align: right; /* Align numbers to the right */
        }
        #results-section table th:first-child, #results-section table td:first-child {
            text-align: left; /* Align variant names to the left */
        }
        #results-section table thead th {
            background-color: #e2e8f0; /* bg-gray-200 */
        }
        #results-section table tfoot td {
            font-weight: bold;
            background-color: #f0f4f8; /* A slightly darker gray for footer */
        }

        /* Stronger border for month separation */
        #main-revenue-table thead th[data-month-header],
        #results-section table thead th[colspan="4"] {
            border-left: 2px solid #94a3b8; /* Stronger, darker border for month headers */
        }
        #main-revenue-table thead th[data-month-header]:first-child,
        #results-section table thead th[colspan="4"]:first-child {
            border-left: none; /* No left border for the very first header */
        }

        /* Apply border to the first data cell of each month's group (Prop. Price) */
        #main-revenue-table tbody tr td[data-column-type="prop-price"],
        #monthly-results-body tr td:nth-child(4n+1) { /* For results table, it's the first of 4 cells */
             border-left: 1px solid #cbd5e0; /* Lighter border for data cells */
        }
        #main-revenue-table tbody tr td[data-column-type="prop-price"]:first-child,
        #monthly-results-body tr td:nth-child(1) {
            border-left: none; /* No left border for the very first prop price cell / first result cell */
        }

        /* Apply border to the first sub-header of each month (Prop. Price) */
        #main-revenue-table thead tr:nth-child(2) th[data-column-type="prop-price"],
        #results-section table thead tr:nth-child(2) th:nth-child(4n+1) {
            border-left: 2px solid #94a3b8; /* Stronger, darker border for the first sub-header of each month */
        }
        #main-revenue-table thead tr:nth-child(2) th[data-column-type="prop-price"]:first-child,
        #results-section table thead tr:nth-child(2) th:first-child {
            border-left: none; /* No left border for the very first sub-header */
        }

        /* Added borders for individual data columns within each month group */
        #main-revenue-table tbody tr td:not([data-column-type="prop-price"]):not(.sticky-col),
        #monthly-results-body tr td:not(:first-child):not([data-column-type="prop-price"]):not(.sticky-col) {
            border-left: 1px solid #e2e8f0; /* Light border between sub-columns */
        }
        #main-revenue-table thead tr:nth-child(2) th:not([data-column-type="prop-price"]):not(:first-child),
        #results-section table thead tr:nth-child(2) th:not(:first-child) {
            border-left: 1px solid #e2e8f0; /* Light border between sub-headers */
        }

        /* Shading for alternate month columns */
        .shaded-month-group {
            background-color: #f3f4f6; /* Tailwind's bg-gray-100 */
        }

        /* Styling for the new "fill-all" input fields */
        .fill-all-input {
            width: 100%; /* Make it fill the header cell */
            padding: 0.25rem 0.5rem;
            font-size: 0.875rem;
            text-align: center;
            border: 1px solid #d1d5db; /* border-gray-300 */
            border-radius: 0.25rem; /* rounded-md */
            margin-top: 0.5rem; /* Space below the text header */
            box-sizing: border-box; /* Include padding and border in the element's total width and height */
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4">
    <div class="bg-white py-8 px-4 rounded-xl shadow-lg w-full">
        <h1 class="text-3xl font-bold text-gray-800 mb-6 text-center">Revenue Change Calculator</h1>

        <div id="input-section" class="space-y-6">
            <!-- Removed the toggle-details-btn div -->
            <!-- Consolidated Table Container -->
            <div class="overflow-x-auto rounded-xl border border-gray-200 shadow-sm">
                <table id="main-revenue-table" class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr id="month-headers-row-1">
                            <!-- Variant header will span 2 rows -->
                            <th rowspan="2" class="px-3 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider sticky-col sticky-col-header">Variant</th>
                            <!-- Month headers will be dynamically added here -->
                        </tr>
                        <tr id="month-headers-row-2">
                            <!-- Sub-headers (Prop. Price, LY Price, Actual Price, Tickets) will be dynamically added here -->
                        </tr>
                        <tr id="fill-all-row">
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider sticky-col sticky-col-header">Fill All:</th>
                            <!-- Fill-all inputs will be dynamically added here -->
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-200" id="table-body">
                        <!-- Variant rows will be dynamically added here -->
                    </tbody>
                </table>
            </div>

            <button id="calculate-btn" class="w-full bg-green-600 text-white py-3 px-6 rounded-xl hover:bg-green-700 transition duration-300 shadow-md">
                Calculate Revenue Change
            </button>
        </div>

        <!-- Results Section -->
        <div id="results-section" class="mt-8 p-6 bg-gray-50 rounded-xl border border-gray-200 hidden">
            <h2 class="text-2xl font-semibold text-gray-800 mb-4 text-center">Detailed Revenue Analysis</h2>

            <h3 class="text-xl font-semibold text-gray-800 mb-3 text-center">Monthly Revenue Breakdown</h3>
            <div class="overflow-x-auto mb-6 rounded-xl border border-gray-200 shadow-sm">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-100">
                        <tr>
                            <th rowspan="2" class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider sticky-col sticky-col-header">Variant</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">September</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">October</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">November</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">December</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">January</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">February</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">March</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">April</th>
                            <th colspan="4" class="px-3 py-2 text-center text-xs font-medium text-gray-700 uppercase tracking-wider border-l border-gray-200">May</th>
                        </tr>
                        <tr>
                            <!-- Sub-headers for each month -->
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>

                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Prop</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">LY</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Act</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Diff (Act-LY)</th>
                        </tr>
                    </thead>
                    <tbody id="monthly-results-body" class="bg-white divide-y divide-gray-200">
                        <!-- Detailed monthly results per variant -->
                    </tbody>
                    <tfoot class="bg-gray-100">
                        <tr>
                            <td class="px-3 py-2 text-sm font-semibold text-gray-900 sticky-col">Monthly Totals</td>
                            <!-- Monthly totals will be dynamically added here -->
                        </tr>
                    </tfoot>
                </table>
            </div>


            <h3 class="text-xl font-semibold text-gray-800 mb-3 text-center">Product Grand Totals</h3>
            <div class="overflow-x-auto mb-6 rounded-xl border border-gray-200 shadow-sm">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-100">
                        <tr>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Variant</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Total LY Revenue</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Total Proposed Revenue</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Total Actual Revenue</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Change (Prop vs LY)</th>
                            <th class="px-3 py-2 text-left text-xs font-medium text-gray-700 uppercase tracking-wider">Change (Act vs LY)</th>
                        </tr>
                    </thead>
                    <tbody id="product-grand-totals-body" class="bg-white divide-y divide-gray-200">
                        <!-- Grand totals per variant -->
                    </tbody>
                </table>
            </div>

            <h3 class="text-xl font-semibold text-gray-800 mb-3 text-center">Overall Grand Totals</h3>
            <div class="space-y-3 text-lg text-gray-700">
                <p><strong>Total Revenue Last Year (All Products):</strong> <span id="grand-ly-revenue" class="font-bold text-blue-600">£0.00</span></p>
                <p><strong>Total Proposed Revenue This Year (All Products):</strong> <span id="grand-prop-revenue" class="font-bold text-purple-600">£0.00</span></p>
                <p><strong>Total Actual Revenue This Year (All Products):</strong> <span id="grand-act-revenue" class="font-bold text-green-600">£0.00</span></p>
                <p><strong>Overall Absolute Change (Proposed vs LY):</strong> <span id="grand-change-prop-abs" class="font-bold text-gray-800">£0.00</span></p>
                <p><strong>Overall Percentage Change (Proposed vs LY):</strong> <span id="grand-change-prop-percent" class="font-bold text-gray-800">0.00%</span></p>
                <p><strong>Overall Absolute Change (Actual vs LY):</strong> <span id="grand-change-act-abs" class="font-bold text-gray-800">£0.00</span></p>
                <p><strong>Overall Percentage Change (Actual vs LY):</strong> <span id="grand-change-act-percent" class="font-bold text-gray-800">0.00%</span></p>
            </div>
            <div id="error-message" class="mt-4 text-red-600 font-medium text-center hidden"></div>
        </div>
    </div>

    <script>
        // JavaScript for the Revenue Change Calculator
        document.addEventListener('DOMContentLoaded', () => {
            const mainRevenueTable = document.getElementById('main-revenue-table');
            const monthHeadersRow1 = document.getElementById('month-headers-row-1');
            const monthHeadersRow2 = document.getElementById('month-headers-row-2');
            const fillAllRow = document.getElementById('fill-all-row'); // New row for fill-all inputs
            const tableBody = document.getElementById('table-body');
            const calculateBtn = document.getElementById('calculate-btn');
            const resultsSection = document.getElementById('results-section');
            const errorMessageDiv = document.getElementById('error-message');
            // Removed the toggleDetailsBtn constant

            // New result display elements
            const monthlyResultsBody = document.getElementById('monthly-results-body');
            const productGrandTotalsBody = document.getElementById('product-grand-totals-body');
            const grandLyRevenueSpan = document.getElementById('grand-ly-revenue');
            const grandPropRevenueSpan = document.getElementById('grand-prop-revenue');
            const grandActRevenueSpan = document.getElementById('grand-act-revenue');
            const grandChangePropAbsSpan = document.getElementById('grand-change-prop-abs');
            const grandChangePropPercentSpan = document.getElementById('grand-change-prop-percent');
            const grandChangeActAbsSpan = document.getElementById('grand-change-act-abs');
            const grandChangeActPercentSpan = document.getElementById('grand-change-act-percent');


            // Array of month names from September to May
            const monthNames = ['September', 'October', 'November', 'December', 'January', 'February', 'March', 'April', 'May'];

            // Pre-defined list of all variants (before filtering)
            const allVariantsRaw = [
                'BASKETBALL', 'DODGEBALL', 'FITNESS GAMES', 'HOCKEY', 'NETBALL', 'RUGBY',
                'TOUCH RUGBY', "WOMEN'S FOOTBALL",
                'Camping Upgrade', 'Extra Player',
                'V.VIP & Deluxe Glamping Package', 'V.VIP & Deluxe Glamping Upgrade',
                'V.VIP & Glamping Package', 'V.VIP & Glamping Upgrade', 'V.VIP Upgrade',
                'VIP & Camping Upgrade', 'VIP Upgrade'
            ];

            // List of individual sports that fall under the "All Sports" category
            const individualSports = [
                'BASKETBALL', 'DODGEBALL', 'FITNESS GAMES', 'HOCKEY', 'NETBALL', 'RUGBY',
                'TOUCH RUGBY', "WOMEN'S FOOTBALL"
            ];

            // Data provided by the user for "Last Year Prices", mapped to a more usable structure
            const userProvidedLastYearPrices = {
                'All Sports Category': { // This represents the group of individual sports
                    'September': 70, 'October': 70,
                    'November': 85, 'December': 85,
                    'January': 99, 'February': 99,
                    'March': 99, 'April': 99, 'May': 0 // Assuming May is 0 for All Sports based on previous data
                },
                'V.VIP & Glamping Upgrade': {
                    'September': 260, 'October': 260, 'November': 275, 'December': 275,
                    'January': 290, 'February': 290, 'March': 300, 'April': 300, 'May': 0
                },
                'VIP & Camping Upgrade': {
                    'September': 140, 'October': 140, 'November': 155, 'December': 155,
                    'January': 175, 'February': 175, 'March': 190, 'April': 190, 'May': 0
                },
                'Camping Upgrade': {
                    'September': 110, 'October': 110, 'November': 125, 'December': 125,
                    'January': 145, 'February': 145, 'March': 155, 'April': 155, 'May': 0
                },
                'VIP Upgrade': {
                    'September': 40, 'October': 40, 'November': 0, 'December': 0, // Explicitly 0 for '-'
                    'January': 50, 'February': 50, 'March': 0, 'April': 0, 'May': 0 // Explicitly 0 for '-'
                },
                'V.VIP Upgrade': {
                    'September': 60, 'October': 60, 'November': 0, 'December': 0, // Explicitly 0 for '-'
                    'January': 70, 'February': 70, 'March': 0, 'April': 0, 'May': 0 // Explicitly 0 for '-'
                }
            };

            // Data provided by the user for "Actual Prices This Year"
            const userProvidedActualPrices = {
                'BASKETBALL': { 'September': 65, 'October': 70, 'November': 85, 'December': 85, 'January': 0, 'February': 99, 'March': 99, 'April': 98, 'May': 0 },
                'DODGEBALL': { 'September': 68, 'October': 70, 'November': 70, 'December': 62, 'January': 79, 'February': 66, 'March': 99, 'April': 99, 'May': 0 },
                'FITNESS GAMES': { 'September': 68, 'October': 70, 'November': 70, 'December': 85, 'January': 79, 'February': 71, 'March': 50, 'April': 81, 'May': 65 },
                'HOCKEY': { 'September': 65, 'October': 70, 'November': 0, 'December': 85, 'January': 96, 'February': 83, 'March': 0, 'April': 0, 'May': 0 },
                'NETBALL': { 'September': 62, 'October': 67, 'November': 85, 'December': 82, 'January': 93, 'February': 99, 'March': 61, 'April': 58, 'May': 0 },
                'RUGBY': { 'September': 61, 'October': 56, 'November': 62, 'December': 82, 'January': 72, 'February': 69, 'March': 77, 'April': 30, 'May': 0 },
                'TOUCH RUGBY': { 'September': 71, 'October': 0, 'November': 0, 'December': 85, 'January': 0, 'February': 65, 'March': 0, 'April': 0, 'May': 0 },
                "WOMEN'S FOOTBALL": { 'September': 0, 'October': 50, 'November': 75, 'December': 85, 'January': 95, 'February': 84, 'March': 0, 'April': 0, 'May': 0 },
                'Camping Upgrade': { 'September': 102, 'October': 102, 'November': 79, 'December': 77, 'January': 98, 'February': 106, 'March': 123, 'April': 74, 'May': 138 },
                'Extra Player': { 'September': 69, 'October': 70, 'November': 36, 'December': 80, 'January': 95, 'February': 87, 'March': 95, 'April': 90, 'May': 82 },
                'V.VIP & Deluxe Glamping Package': { 'September': 0, 'October': 0, 'November': 0, 'December': 0, 'January': 350, 'February': 0, 'March': 0, 'April': 0, 'May': 0 },
                'V.VIP & Deluxe Glamping Upgrade': { 'September': 0, 'October': 0, 'November': 0, 'December': 0, 'January': 0, 'February': 0, 'March': 360, 'April': 360, 'May': 360 },
                'V.VIP & Glamping Package': { 'September': 250, 'October': 260, 'November': 275, 'December': 275, 'January': 266, 'February': 246, 'March': 0, 'April': 0, 'May': 0 },
                'V.VIP & Glamping Upgrade': { 'September': 252, 'October': 263, 'November': 275, 'December': 275, 'January': 293, 'February': 269, 'March': 268, 'April': 256, 'May': 300 },
                'V.VIP Upgrade': { 'September': 28, 'October': 44, 'November': 70, 'December': 54, 'January': 70, 'February': 49, 'March': 64, 'April': 33, 'May': 51 },
                'VIP & Camping Upgrade': { 'September': 127, 'October': 111, 'November': 65, 'December': 91, 'January': 123, 'February': 101, 'March': 155, 'April': 138, 'May': 190 },
                'VIP Upgrade': { 'September': 31, 'October': 40, 'November': 40, 'December': 40, 'January': 0, 'February': 43, 'March': 21, 'April': 41, 'May': 14 }
            };

            // Data provided by the user for "Tickets Sold" - MERGED DATA
            const userProvidedTicketsSold = {
                "RUGBY": { "September": 31, "October": 30, "November": 39, "December": 20, "January": 16, "February": 24, "March": 52, "April": 11, "May": 10 },
                "NETBALL": { "September": 23, "October": 30, "November": 20, "December": 30, "January": 66, "February": 11, "March": 30, "April": 12, "May": 0 },
                "DODGEBALL": { "September": 65, "October": 75, "November": 12, "December": 17, "January": 64, "February": 88, "March": 21, "April": 0, "May": 0 },
                "HOCKEY": { "September": 93, "October": 41, "November": 0, "December": 24, "January": 50, "February": 64, "March": 0, "April": 79, "May": 10 },
                "FITNESS GAMES": { "September": 0, "October": 0, "November": 0, "December": 11, "January": 0, "February": 0, "March": 0, "April": 0, "May": 0 },
                "BASKETBALL": { "September": 20, "October": 16, "November": 4, "December": 13, "January": 0, "February": 44, "March": 0, "April": 0, "May": 0 },
                "TOUCH RUGBY": { "September": 0, "October": 0, "November": 0, "December": 0, "January": 0, "February": 0, "March": 0, "April": 0, "May": 0 },
                "WOMEN'S FOOTBALL": { "September": 0, "October": 0, "November": 0, "December": 0, "January": 0, "February": 0, "March": 0, "April": 1, "May": 0 },
                "V.VIP & Glamping Package": { "September": 30, "October": 40, "November": 0, "December": 0, "January": 0, "February": 0, "March": 0, "April": 0, "May": 0 },
                "V.VIP Upgrade": { "September": 41, "October": 14, "November": 11, "December": 10, "January": 1, "February": 37, "March": 23, "April": 17, "May": 15 },
                "Camping Upgrade": { "September": 82, "October": 90, "November": 24, "December": 92, "January": 50, "February": 30, "March": 15, "April": 43, "May": 6 },
                "VIP Upgrade": { "September": 34, "October": 0, "November": 29, "December": 1, "January": 14, "February": 37, "March": 63, "April": 30, "May": 30 },
                "Extra Player": { "September": 11, "October": 57, "November": 23, "December": 100, "January": 21, "February": 71, "March": 42, "April": 43, "May": 13 },
                "VIP & Camping Upgrade": { "September": 12, "October": 85, "November": 0, "December": 31, "January": 11, "February": 13, "March": 7, "April": 19, "May": 2 },
                "V.VIP & Glamping Upgrade": { "September": 0, "October": 30, "November": 5, "December": 3, "January": 1, "February": 31, "March": 25, "April": 30, "May": 1 },
                "V.VIP & Deluxe Glamping Upgrade": { "September": 0, 'October': 0, 'November': 0, 'December': 0, 'January': 0, 'February': 0, 'March': 3, 'April': 4, 'May': 3 }
            };


            // Initialize previousYearData with all variants and months, defaulting to 0 for all prices and tickets
            const previousYearData = {};
            monthNames.forEach(month => {
                previousYearData[month] = [];
                allVariantsRaw.forEach(variant => {
                    previousYearData[month].push({ variant: variant, lastYearPrice: 0, actualPriceThisYear: 0, ticketsSold: 0 }); // Default all to 0
                });
            });

            // Overlay the user-provided Last Year Prices onto the initialized structure
            for (const categoryOrVariant in userProvidedLastYearPrices) {
                if (categoryOrVariant === 'All Sports Category') {
                    for (const month in userProvidedLastYearPrices[categoryOrVariant]) {
                        const price = userProvidedLastYearPrices[categoryOrVariant][month];
                        individualSports.forEach(sportVariant => {
                            const monthEntry = previousYearData[month].find(item => item.variant === sportVariant);
                            if (monthEntry) {
                                monthEntry.lastYearPrice = price;
                            }
                        });
                    }
                } else {
                    for (const month in userProvidedLastYearPrices[categoryOrVariant]) {
                        const price = userProvidedLastYearPrices[categoryOrVariant][month];
                        const monthEntry = previousYearData[month].find(item => item.variant === categoryOrVariant);
                        if (monthEntry) {
                            monthEntry.lastYearPrice = price;
                        }
                    }
                }
            }

            // Overlay the user-provided Actual Prices onto the initialized structure
            for (const variantName in userProvidedActualPrices) {
                for (const month in userProvidedActualPrices[variantName]) {
                    const price = userProvidedActualPrices[variantName][month];
                    const monthEntry = previousYearData[month].find(item => item.variant === variantName);
                    if (monthEntry) {
                        monthEntry.actualPriceThisYear = price;
                    }
                }
            }

            // Overlay the user-provided Tickets Sold onto the initialized structure
            for (const variantName in userProvidedTicketsSold) {
                for (const month in userProvidedTicketsSold[variantName]) {
                    const tickets = userProvidedTicketsSold[variantName][month];
                    const monthEntry = previousYearData[month].find(item => item.variant === variantName);
                    if (monthEntry) {
                        monthEntry.ticketsSold = tickets;
                    }
                }
            }

            // Filter allVariantsRaw to only include those with at least one non-zero lastYearPrice
            const allVariants = allVariantsRaw.filter(variantName => {
                let hasNonZeroPrice = false;
                for (const month of monthNames) {
                    const variantDataForMonth = previousYearData[month].find(v => v.variant === variantName);
                    if (variantDataForMonth && variantDataForMonth.lastYearPrice > 0) {
                        hasNonZeroPrice = true;
                        break;
                    }
                }
                return hasNonZeroPrice;
            });


            /**
             * Initializes the main revenue table with headers and variant rows.
             */
            const initializeTable = () => {
                // Clear existing content
                monthHeadersRow1.innerHTML = `<th rowspan="2" class="px-3 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider sticky-col sticky-col-header">Variant</th>`;
                monthHeadersRow2.innerHTML = ``; // This row starts empty after the variant header is set in row1
                fillAllRow.innerHTML = `<th class="px-3 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider sticky-col sticky-col-header">Fill All:</th>`; // Clear fill-all row
                tableBody.innerHTML = '';

                // Add month headers (first row)
                monthNames.forEach((month, index) => {
                    const th = document.createElement('th');
                    // Set colspan to 1 as only Prop. Price is visible by default
                    th.setAttribute('colspan', '1');
                    th.classList.add('px-3', 'py-2', 'text-center', 'text-xs', 'font-medium', 'text-gray-500', 'uppercase', 'tracking-wider', 'border-l', 'border-gray-200');
                    if (index % 2 === 0) { // Apply shading to September, November, January, etc.
                        th.classList.add('shaded-month-group');
                    }
                    th.textContent = month;
                    th.setAttribute('data-month-header', month); // Add data attribute to easily select month headers
                    monthHeadersRow1.appendChild(th);
                });

                // Add sub-headers (second row) - ORDERED Prop. Price, LY Price, Act. Price, Tickets
                const subHeaderTypes = ['prop-price', 'ly-price', 'act-price', 'tickets'];
                const subHeaderTexts = {
                    'prop-price': 'Prop. Price',
                    'ly-price': 'LY Price',
                    'act-price': 'Act. Price',
                    'tickets': 'Tickets'
                };

                monthNames.forEach((month, monthIndex) => {
                    subHeaderTypes.forEach(type => {
                        const subTh = document.createElement('th');
                        subTh.classList.add('px-3', 'py-2', 'text-left', 'text-xs', 'font-medium', 'text-gray-500', 'uppercase', 'tracking-wider');
                        if (type !== 'prop-price') { // Hide all except 'prop-price' by default
                            subTh.classList.add('hidden');
                        }
                        if (monthIndex % 2 === 0) { // Apply shading to September, November, January, etc.
                            subTh.classList.add('shaded-month-group');
                        }
                        subTh.textContent = subHeaderTexts[type];
                        subTh.setAttribute('data-column-type', type);
                        subTh.setAttribute('data-month', month); // Add month to sub-header for specific toggling
                        monthHeadersRow2.appendChild(subTh);
                    });
                });

                // Add fill-all input fields for each sub-column
                monthNames.forEach((month, index) => {
                    subHeaderTypes.forEach(type => {
                        const fillAllSubCell = document.createElement('th');
                        fillAllSubCell.classList.add('px-3', 'py-2', 'text-left', 'text-xs', 'font-medium', 'text-gray-500', 'uppercase', 'tracking-wider');

                        if (index % 2 === 0) {
                            fillAllSubCell.classList.add('shaded-month-group');
                        }
                        if (type !== 'prop-price') { // Hide all except 'prop-price' by default
                            fillAllSubCell.classList.add('hidden');
                        }

                        if (type === 'prop-price') {
                            const propPriceFillAllInput = document.createElement('input');
                            propPriceFillAllInput.type = 'number';
                            propPriceFillAllInput.classList.add('fill-all-input');
                            propPriceFillAllInput.placeholder = 'Fill all...';
                            propPriceFillAllInput.min = '0';
                            propPriceFillAllInput.step = '0.01';
                            propPriceFillAllInput.setAttribute('data-fill-month', month);
                            fillAllSubCell.appendChild(propPriceFillAllInput);
                        } else {
                            fillAllSubCell.innerHTML = `&nbsp;`; // Use non-breaking space to maintain cell height
                        }
                        fillAllRow.appendChild(fillAllSubCell);
                    });
                });


                // Populate table body with variant rows using the FILTERED allVariants
                allVariants.forEach(variantName => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50';

                    // Variant name cell (sticky)
                    const variantNameCell = document.createElement('td');
                    variantNameCell.className = 'px-3 py-2 whitespace-nowrap text-sm font-medium text-gray-900 sticky-col';
                    variantNameCell.textContent = variantName;
                    row.appendChild(variantNameCell);

                    // Add cells for each month (ORDERED Prop. Price, LY Price, Act. Price, Tickets)
                    monthNames.forEach((month, monthIndex) => {
                        const variantDataForMonth = previousYearData[month] ? previousYearData[month].find(v => v.variant === variantName) : null;
                        const lastYearPrice = variantDataForMonth ? variantDataForMonth.lastYearPrice : 0;
                        const actualPriceThisYear = variantDataForMonth ? variantDataForMonth.actualPriceThisYear : 0;
                        const ticketsSold = variantDataForMonth ? variantDataForMonth.ticketsSold : 0;

                        // Proposed Price This Year (editable) - FIRST
                        const propPriceCell = document.createElement('td');
                        propPriceCell.classList.add('px-3', 'py-2', 'whitespace-nowrap');
                        if (monthIndex % 2 === 0) {
                            propPriceCell.classList.add('shaded-month-group');
                        }
                        // Removed placeholder attribute
                        propPriceCell.innerHTML = `<input type="number" class="variant-input prop-price-input" min="0" step="0.01" data-variant="${variantName}" data-month="${month}">`;
                        propPriceCell.setAttribute('data-column-type', 'prop-price');
                        row.appendChild(propPriceCell);

                        // Last Year Price (readonly) - SECOND, HIDDEN BY DEFAULT
                        const lyPriceCell = document.createElement('td');
                        lyPriceCell.classList.add('px-3', 'py-2', 'whitespace-nowrap', 'hidden'); // Hidden by default
                        if (monthIndex % 2 === 0) {
                            lyPriceCell.classList.add('shaded-month-group');
                        }
                        lyPriceCell.innerHTML = `<input type="number" class="variant-input ly-price-input" value="${lastYearPrice.toFixed(2)}" readonly min="0" step="0.01" data-variant="${variantName}" data-month="${month}">`;
                        lyPriceCell.setAttribute('data-column-type', 'ly-price');
                        row.appendChild(lyPriceCell);

                        // Actual Price This Year (hardcoded and readonly) - THIRD, HIDDEN BY DEFAULT
                        const actPriceCell = document.createElement('td');
                        actPriceCell.classList.add('px-3', 'py-2', 'whitespace-nowrap', 'hidden'); // Hidden by default
                        if (monthIndex % 2 === 0) {
                            actPriceCell.classList.add('shaded-month-group');
                        }
                        actPriceCell.innerHTML = `<input type="number" class="variant-input actual-price-input" value="${actualPriceThisYear.toFixed(2)}" readonly min="0" step="0.01" data-variant="${variantName}" data-month="${month}">`;
                        actPriceCell.setAttribute('data-column-type', 'act-price');
                        row.appendChild(actPriceCell);

                        // Tickets Sold (hardcoded and readonly) - FOURTH, HIDDEN BY DEFAULT
                        const ticketsSoldCell = document.createElement('td');
                        ticketsSoldCell.classList.add('px-3', 'py-2', 'whitespace-nowrap', 'hidden'); // Hidden by default
                        if (monthIndex % 2 === 0) {
                            ticketsSoldCell.classList.add('shaded-month-group');
                        }
                        ticketsSoldCell.innerHTML = `<input type="number" class="variant-input tickets-sold-input" value="${ticketsSold}" readonly min="0" step="1" data-variant="${variantName}" data-month="${month}">`;
                        ticketsSoldCell.setAttribute('data-column-type', 'tickets');
                        row.appendChild(ticketsSoldCell);
                    });
                    tableBody.appendChild(row);
                });

                // Add event listeners to the new fill-all inputs
                fillAllRow.querySelectorAll('.fill-all-input').forEach(input => {
                    input.addEventListener('input', (event) => {
                        const monthToFill = event.target.dataset.fillMonth;
                        const valueToFill = event.target.value;

                        // Select all 'prop-price-input' fields for the specific month
                        // and filter them to only include sports variants
                        const monthSpecificPropPriceInputs = tableBody.querySelectorAll(`.prop-price-input[data-month="${monthToFill}"]`);

                        monthSpecificPropPriceInputs.forEach(propInput => {
                            const variantName = propInput.dataset.variant;
                            if (individualSports.includes(variantName)) { // Only fill for sports variants
                                propInput.value = valueToFill;
                            }
                        });
                    });
                });
            };

            /**
             * Toggles the visibility of LY Price, Actual Price, and Tickets columns, and adjusts fill-all inputs.
             * This function is now only called by the toggleDetailsBtn, which is removed.
             * However, keeping it in case future functionality needs similar toggling.
             */
            // Removed the toggleColumns function as it's no longer needed

            /**
             * Calculates and displays the revenue change.
             */
            const calculateRevenueChange = () => {
                let totalLastYearRevenueOverall = 0;
                let totalProposedRevenueOverall = 0;
                let totalActualRevenueOverall = 0;
                let hasError = false;
                errorMessageDiv.classList.add('hidden'); // Hide previous errors

                // Clear previous results
                monthlyResultsBody.innerHTML = '';
                productGrandTotalsBody.innerHTML = '';

                let monthlyTotals = {}; // { 'September': { ly: 0, prop: 0, act: 0 }, ... }
                monthNames.forEach(month => {
                    monthlyTotals[month] = { ly: 0, prop: 0, act: 0 };
                });

                // Iterate over each variant row
                allVariants.forEach(variantName => { // Use the FILTERED allVariants
                    let productLyTotal = 0;
                    let productPropTotal = 0;
                    let productActTotal = 0;

                    // Create a row for the monthly breakdown table
                    const monthlyTableRow = document.createElement('tr');
                    monthlyTableRow.className = 'hover:bg-gray-50';
                    monthlyTableRow.innerHTML = `<td class="px-3 py-2 whitespace-nowrap text-sm font-medium text-gray-900 sticky-col">${variantName}</td>`;

                    // Find the row for this variant in the input table
                    let currentRow = null;
                    const allInputRows = tableBody.querySelectorAll('tr');
                    for (const r of allInputRows) {
                        if (r.querySelector('td.sticky-col').textContent === variantName) {
                            currentRow = r;
                            break;
                        }
                    }

                    if (!currentRow) {
                        console.error(`Input row not found for variant: ${variantName}`);
                        hasError = true; // Set error flag
                        errorMessageDiv.textContent = `Internal error: Input row not found for variant: ${variantName}.`;
                        errorMessageDiv.classList.remove('hidden');
                        return; // Skip this variant if its input row isn't found
                    }

                    // Iterate over each month's inputs within this variant's row
                    monthNames.forEach((month, monthIndex) => {
                        // Select inputs for the current variant and month (order matches input table)
                        const propPriceInput = currentRow.querySelector(`.prop-price-input[data-variant="${variantName}"][data-month="${month}"]`);
                        const lyPriceInput = currentRow.querySelector(`.ly-price-input[data-variant="${variantName}"][data-month="${month}"]`);
                        const actualPriceInput = currentRow.querySelector(`.actual-price-input[data-variant="${variantName}"][data-month="${month}"]`);
                        const ticketsSoldInput = currentRow.querySelector(`.tickets-sold-input[data-variant="${variantName}"][data-month="${month}"]`);

                        // Check if inputs exist (they should if table is correctly rendered)
                        if (!propPriceInput || !lyPriceInput || !actualPriceInput || !ticketsSoldInput) {
                            console.error(`Missing inputs for ${variantName} in ${month}`);
                            hasError = true;
                            errorMessageDiv.textContent = `Internal error: Missing data for ${variantName} in ${month}.`;
                            errorMessageDiv.classList.remove('hidden');
                            return;
                        }

                        const lastYearPrice = parseFloat(lyPriceInput.value);
                        // Use proposedPriceInput.value directly for proposedPriceThisYear
                        const proposedPriceThisYear = parseFloat(propPriceInput.value);
                        const actualPriceThisYear = parseFloat(actualPriceInput.value);
                        const ticketsSold = parseInt(ticketsSoldInput.value);

                        // Validate inputs
                        if (isNaN(lastYearPrice) || isNaN(proposedPriceThisYear) || isNaN(actualPriceThisYear) || isNaN(ticketsSold) ||
                            lastYearPrice < 0 || proposedPriceThisYear < 0 || actualPriceThisYear < 0 || ticketsSold < 0) {
                            hasError = true;
                            errorMessageDiv.textContent = `Please enter valid positive numbers for all fields for "${variantName}" in "${month}".`;
                            errorMessageDiv.classList.remove('hidden');
                            return; // Exit inner loop if error found
                        }

                        const monthlyLyRevenue = lastYearPrice * ticketsSold;
                        const monthlyPropRevenue = proposedPriceThisYear * ticketsSold;
                        const monthlyActRevenue = actualPriceThisYear * ticketsSold;
                        const monthlyDiffActLy = monthlyActRevenue - monthlyLyRevenue;

                        // Accumulate monthly totals
                        monthlyTotals[month].ly += monthlyLyRevenue;
                        monthlyTotals[month].prop += monthlyPropRevenue;
                        monthlyTotals[month].act += monthlyActRevenue;

                        // Accumulate product totals
                        productLyTotal += monthlyLyRevenue;
                        productPropTotal += monthlyPropRevenue;
                        productActTotal += monthlyActRevenue;

                        // Add cells to the monthly breakdown row (ORDERED PROP, LY, ACT, DIFF)
                        monthlyTableRow.innerHTML += `
                            <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-700 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${monthlyPropRevenue.toFixed(2)}</td>
                            <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-700 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${monthlyLyRevenue.toFixed(2)}</td>
                            <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-700 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${monthlyActRevenue.toFixed(2)}</td>
                            <td class="px-3 py-2 whitespace-nowrap text-sm ${monthlyDiffActLy > 0 ? 'text-green-600' : (monthlyDiffActLy < 0 ? 'text-red-600' : '')} ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${monthlyDiffActLy.toFixed(2)}</td>
                        `;
                    });

                    if (hasError) {
                        return; // Exit outer loop if error found
                    }
                    monthlyResultsBody.appendChild(monthlyTableRow);

                    // Add row to product grand totals table
                    const productGrandRow = document.createElement('tr');
                    const productChangePropLy = productPropTotal - productLyTotal;
                    const productChangeActLy = productActTotal - productLyTotal;

                    productGrandRow.innerHTML = `
                        <td class="px-3 py-2 whitespace-nowrap text-sm font-medium text-gray-900">${variantName}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm text-blue-600 font-bold">£${productLyTotal.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm text-purple-600 font-bold">£${productPropTotal.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm text-green-600 font-bold">£${productActTotal.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm ${productChangePropLy > 0 ? 'text-green-600' : (productChangePropLy < 0 ? 'text-red-600' : '')}">£${productChangePropLy.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm ${productChangeActLy > 0 ? 'text-green-600' : (productChangeActLy < 0 ? 'text-red-600' : '')}">£${productChangeActLy.toFixed(2)}</td>
                    `;
                    productGrandTotalsBody.appendChild(productGrandRow);

                    // Accumulate overall grand totals
                    totalLastYearRevenueOverall += productLyTotal;
                    totalProposedRevenueOverall += productPropTotal;
                    totalActualRevenueOverall += productActTotal;
                });

                if (hasError) {
                    resultsSection.classList.add('hidden');
                    return; // Stop if there was an input error
                }

                // Populate monthly totals row in the footer of the monthly breakdown table
                const monthlyTotalsFooterRow = document.querySelector('#monthly-results-body').nextElementSibling.querySelector('tr');
                // Clear previous monthly totals to prevent duplication on recalculate
                monthlyTotalsFooterRow.innerHTML = `<td class="px-3 py-2 text-sm font-semibold text-gray-900 sticky-col">Monthly Totals</td>`;

                monthNames.forEach((month, monthIndex) => {
                    const ly = monthlyTotals[month].ly;
                    const prop = monthlyTotals[month].prop;
                    const act = monthlyTotals[month].act;
                    const diffActLy = act - ly;

                    monthlyTotalsFooterRow.innerHTML += `
                        <td class="px-3 py-2 whitespace-nowrap text-sm font-semibold text-gray-900 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${prop.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm font-semibold text-gray-900 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${ly.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm font-semibold text-gray-900 ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${act.toFixed(2)}</td>
                        <td class="px-3 py-2 whitespace-nowrap text-sm font-semibold ${diffActLy > 0 ? 'text-green-600' : (diffActLy < 0 ? 'text-red-600' : '')} ${monthIndex % 2 === 0 ? 'shaded-month-group' : ''}">£${diffActLy.toFixed(2)}</td>
                    `;
                });


                // Calculate overall percentage changes
                const overallChangePropAbs = totalProposedRevenueOverall - totalLastYearRevenueOverall;
                let overallChangePropPercent = 0;
                if (totalLastYearRevenueOverall !== 0) {
                    overallChangePropPercent = (overallChangePropAbs / totalLastYearRevenueOverall) * 100;
                } else if (totalProposedRevenueOverall > 0) {
                    overallChangePropPercent = Infinity;
                }

                const overallChangeActAbs = totalActualRevenueOverall - totalLastYearRevenueOverall;
                let overallChangeActPercent = 0;
                if (totalLastYearRevenueOverall !== 0) {
                    overallChangeActPercent = (overallChangeActAbs / totalLastYearRevenueOverall) * 100;
                } else if (totalActualRevenueOverall > 0) {
                    overallChangeActPercent = Infinity;
                }

                // Display overall grand totals
                grandLyRevenueSpan.textContent = `£${totalLastYearRevenueOverall.toFixed(2)}`;
                grandPropRevenueSpan.textContent = `£${totalProposedRevenueOverall.toFixed(2)}`;
                grandActRevenueSpan.textContent = `£${totalActualRevenueOverall.toFixed(2)}`;

                grandChangePropAbsSpan.textContent = `£${overallChangePropAbs.toFixed(2)}`;
                if (overallChangePropPercent === Infinity) {
                    grandChangePropPercentSpan.textContent = 'Infinite Increase';
                    grandChangePropPercentSpan.className = 'font-bold text-green-600';
                } else {
                    grandChangePropPercentSpan.textContent = `${overallChangePropPercent.toFixed(2)}%`;
                    grandChangePropPercentSpan.className = `font-bold ${overallChangePropPercent > 0 ? 'text-green-600' : (overallChangePropPercent < 0 ? 'text-red-600' : 'text-gray-800')}`;
                }

                grandChangeActAbsSpan.textContent = `£${overallChangeActAbs.toFixed(2)}`;
                if (overallChangeActPercent === Infinity) {
                    grandChangeActPercentSpan.textContent = 'Infinite Increase';
                    grandChangeActPercentSpan.className = 'font-bold text-green-600';
                } else {
                    grandChangeActPercentSpan.textContent = `${overallChangeActPercent.toFixed(2)}%`;
                    grandChangeActPercentSpan.className = `font-bold ${overallChangeActPercent > 0 ? 'text-green-600' : (overallChangeActPercent < 0 ? 'text-red-600' : 'text-gray-800')}`;
                }

                resultsSection.classList.remove('hidden'); // Show results section
            };

            // Event Listeners
            calculateBtn.addEventListener('click', calculateRevenueChange);
            // Removed the event listener for toggleDetailsBtn

            // Initialize the table on page load
            initializeTable();
        });
    </script>
</body>
</html>
