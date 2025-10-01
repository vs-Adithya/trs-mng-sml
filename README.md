<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Transaction Management Simulator</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');

  body {
    font-family: 'Inter', sans-serif;
    background-color: #f0f4f8;
    margin: 20px;
    color: #222;
  }
  h1, h2 {
    text-align: center;
    color: #2c3e50;
    font-weight: 600;
  }
  section {
    background: white;
    border-radius: 12px;
    padding: 20px;
    max-width: 900px;
    margin: 30px auto;
    box-shadow: 0 8px 20px rgb(0 0 0 / 0.1);
  }
  .btn-group {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-bottom: 15px;
    flex-wrap: wrap;
  }
  button {
    background-color: #2980b9;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 12px 18px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: 600;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }
  button:hover:not(:disabled) {
    background-color: #1c5980;
    transform: translateY(-2px);
  }
  button:disabled {
    background-color: #a0aec0;
    cursor: not-allowed;
    transform: none;
  }
  .state-box {
    max-width: 400px;
    margin: 20px auto;
    background: #eef5fc;
    border-radius: 16px;
    padding: 20px;
    text-align: center;
    font-weight: 600;
    font-size: 1.3rem;
    color: #2c3e50;
    box-shadow: 0 4px 15px rgb(41 128 185 / 0.3);
    transition: background-color 0.5s ease;
  }
  .state-active {background-color: #d1e7dd; color:#0f5132;}
  .state-partial {background-color: #ffe5d9; color:#994026;}
  .state-committed {background-color: #b6f0b1; color:#1e4620;}
  .state-failed {background-color: #f8d7da; color:#842029;}
  .state-aborted {background-color: #f5c6cb; color:#6b0a0a;}

  /* ACID properties*/
  .acid-container {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
    justify-content: center;
  }
  .acid-card {
    background: #dbeafe;
    border-radius: 12px;
    flex: 1 1 180px;
    padding: 18px;
    box-shadow: 0 4px 10px rgb(66 153 225 / 0.3);
  }
  .acid-card h3 {
    margin-bottom: 10px;
    color: #1e40af;
  }
  .acid-card p {
    color: #1e293b;
    font-size: 0.95rem;
  }

  /* Dirty Read Simulation */
  .transaction-table {
    width: 100%;
    border-collapse: collapse;
    margin: 10px 0 30px 0;
  }
  .transaction-table th, .transaction-table td {
    border: 1px solid #ddd;
    padding: 8px 12px;
    text-align: center;
  }
  .transaction-table th {
    background-color: #2980b9;
    color: white;
  }
  .log-container {
    max-height: 120px;
    overflow-y: auto;
    background: #fefefe;
    border: 1px solid #bbb;
    padding: 10px;
    font-family: monospace;
    font-size: 0.9rem;
    border-radius: 8px;
  }

  /* Privileges demo */
  .privilege-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
  }
  .privilege-table th, .privilege-table td {
    border: 1px solid #666;
    padding: 10px;
    text-align: center;
    font-weight: 600;
  }
  .privilege-table th {
    background-color: #4a90e2;
    color: white;
  }

</style>
</head>
<body>

<h1>Transaction Management Simulator</h1>

<section aria-labelledby="transaction-states-title">
  <h2 id="transaction-states-title">Transaction States</h2>
  <div class="btn-group" role="group" aria-label="Transaction control buttons">
    <button id="start-btn">Start Transaction</button>
    <button id="partial-btn" disabled>Partially Committed</button>
    <button id="commit-btn" disabled>Commit</button>
    <button id="fail-btn" disabled>Fail</button>
    <button id="abort-btn" disabled>Abort</button>
  </div>
  <div id="tx-state-display" class="state-box state-active" role="alert" aria-live="polite">Active</div>
</section>

<section aria-labelledby="acid-properties-title">
  <h2 id="acid-properties-title">ACID Properties Explained</h2>
  <div class="acid-container">
    <div class="acid-card" tabindex="0">
      <h3>Atomicity</h3>
      <p>Ensures that all operations in a transaction are completed; if not, the transaction is aborted and no changes are made.</p>
    </div>
    <div class="acid-card" tabindex="0">
      <h3>Consistency</h3>
      <p>The database moves from one consistent state to another, maintaining all rules and constraints.</p>
    </div>
    <div class="acid-card" tabindex="0">
      <h3>Isolation</h3>
      <p>Concurrent transactions do not interfere with each other, ensuring isolated execution.</p>
    </div>
    <div class="acid-card" tabindex="0">
      <h3>Durability</h3>
      <p>Once committed, changes are permanent even in the event of system failures.</p>
    </div>
  </div>
</section>

<section aria-labelledby="dirty-read-title">
  <h2 id="dirty-read-title">Dirty Read Scenario Simulation</h2>
  <p>Two transactions interact with shared data illustrating a dirty read.</p>
  <table class="transaction-table" aria-label="Shared account balance">
    <thead>
      <tr><th>Account</th><th>Balance</th></tr>
    </thead>
    <tbody>
      <tr><td>Checking</td><td id="balance">1000</td></tr>
    </tbody>
  </table>

  <div class="btn-group" role="group" aria-label="Dirty read operation buttons">
    <button id="tx1-update">Transaction 1: Update Balance to 500</button>
    <button id="tx2-read">Transaction 2: Read Balance</button>
    <button id="tx1-abort">Transaction 1: Abort</button>
  </div>

  <h3>Log</h3>
  <div id="dirty-read-log" class="log-container" aria-live="polite" role="log" tabindex="0"></div>
</section>

<section aria-labelledby="privileges-title">
  <h2 id="privileges-title">GRANT and REVOKE Privileges Demo</h2>
  <p>Manage user privileges on a sample table.</p>
  <table class="privilege-table" aria-label="User privileges table">
    <thead>
      <tr><th>User</th><th>Privileges</th><th>Actions</th></tr>
    </thead>
    <tbody id="privileges-tbody">
      <tr data-user="Alice"><td>Alice</td><td class="privs">SELECT</td><td><button class="grant-btn" data-user="Alice">GRANT INSERT</button> <button class="revoke-btn" data-user="Alice">REVOKE SELECT</button></td></tr>
      <tr data-user="Bob"><td>Bob</td><td class="privs">SELECT, INSERT</td><td><button class="grant-btn" data-user="Bob">GRANT UPDATE</button> <button class="revoke-btn" data-user="Bob">REVOKE INSERT</button></td></tr>
    </tbody>
  </table>
</section>

<script>
  // Transaction States Simulation
  const stateDisplay = document.getElementById('tx-state-display');
  const btnStart = document.getElementById('start-btn');
  const btnPartial = document.getElementById('partial-btn');
  const btnCommit = document.getElementById('commit-btn');
  const btnFail = document.getElementById('fail-btn');
  const btnAbort = document.getElementById('abort-btn');

  let txState = 'Active';

  function updateState(newState) {
    txState = newState;
    const statesClassMap = {
      'Active': 'state-active',
      'Partially Committed': 'state-partial',
      'Committed': 'state-committed',
      'Failed': 'state-failed',
      'Aborted': 'state-aborted'
    };
    stateDisplay.textContent = newState;
    for(const cls of Object.values(statesClassMap)) {
      stateDisplay.classList.remove(cls);
    }
    stateDisplay.classList.add(statesClassMap[newState]);

    btnStart.disabled = (newState !== 'Active');
    btnPartial.disabled = (newState !== 'Active');
    btnCommit.disabled = (newState !== 'Partially Committed');
    btnFail.disabled = (newState !== 'Active' && newState !== 'Partially Committed');
    btnAbort.disabled = (newState !== 'Failed');

  }

  btnStart.addEventListener('click', () => updateState('Active'));
  btnPartial.addEventListener('click', () => updateState('Partially Committed'));
  btnCommit.addEventListener('click', () => updateState('Committed'));
  btnFail.addEventListener('click', () => updateState('Failed'));
  btnAbort.addEventListener('click', () => updateState('Aborted'));

  updateState('Active');

  // Dirty Read Simulation
  const balanceTd = document.getElementById('balance');
  const tx1UpdateBtn = document.getElementById('tx1-update');
  const tx2ReadBtn = document.getElementById('tx2-read');
  const tx1AbortBtn = document.getElementById('tx1-abort');
  const dirtyLog = document.getElementById('dirty-read-log');

  let actualBalance = 1000;
  let tx1Balance = null;
  let tx1Active = false;

  function logMessage(msg) {
    dirtyLog.textContent += msg + '\n';
    dirtyLog.scrollTop = dirtyLog.scrollHeight;
  }

  tx1UpdateBtn.addEventListener('click', () => {
    if(tx1Active) {
      logMessage('Transaction 1 already active and uncommitted.');
      return;
    }
    tx1Balance = 500;
    tx1Active = true;
    balanceTd.textContent = tx1Balance + ' (Uncommitted by Tx1)';
    logMessage('Transaction 1 updates balance to 500 but NOT committed yet.');
  });

  tx2ReadBtn.addEventListener('click', () => {
    const readBalance = tx1Active ? tx1Balance : actualBalance;
    logMessage(`Transaction 2 reads balance: ${readBalance} (Dirty Read occurs if Tx1 not committed)`);
    alert(`Transaction 2 reads balance: ${readBalance} (Dirty Read scenario)`);
  });

  tx1AbortBtn.addEventListener('click', () => {
    if(!tx1Active) {
      logMessage('No active Transaction 1 to abort.');
      return;
    }
    tx1Active = false;
    tx1Balance = null;
    balanceTd.textContent = actualBalance;
    logMessage('Transaction 1 aborted. Changes discarded.');
  });

  // GRANT and REVOKE Privileges Demo
  const privilegesTable = document.getElementById('privileges-tbody');

  privilegesTable.addEventListener('click', (e) => {
    if(e.target.matches('button.grant-btn')) {
      const user = e.target.dataset.user;
      const privilege = e.target.textContent.split(' ')[1];
      grantPrivilege(user, privilege);
    } else if(e.target.matches('button.revoke-btn')) {
      const user = e.target.dataset.user;
      const privilege = e.target.textContent.split(' ')[1];
      revokePrivilege(user, privilege);
    }
  });

  function grantPrivilege(user, privilege) {
    const row = privilegesTable.querySelector(`tr[data-user="${user}"]`);
    const privCell = row.querySelector('.privs');
    let privs = privCell.textContent.split(',').map(p => p.trim());
    if(!privs.includes(privilege)) {
      privs.push(privilege);
      privCell.textContent = privs.sort().join(', ');
      alert(`Granted ${privilege} to ${user}.`);
    } else {
      alert(`${user} already has ${privilege} privilege.`);
    }
  }

  function revokePrivilege(user, privilege) {
    const row = privilegesTable.querySelector(`tr[data-user="${user}"]`);
    const privCell = row.querySelector('.privs');
    let privs = privCell.textContent.split(',').map(p => p.trim());
    if(privs.includes(privilege)) {
      privs = privs.filter(p => p !== privilege);
      privCell.textContent = privs.length ? privs.join(', ') : 'None';
      alert(`Revoked ${privilege} from ${user}.`);
    } else {
      alert(`${user} does not have ${privilege} privilege.`);
    }
  }
</script>

</body>
</html>
