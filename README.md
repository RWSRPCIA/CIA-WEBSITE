<script>
let currentUser = '';
let users = {
  'soar': {
    password: 'sasrp',
    role: 'agent',
    badge: 'CIA-2847',
    department: 'Wildlife Enforcement',
    yearsOfService: 3,
    rank: 'Officer'
  },
  'fanzo': {
    password: 'sasrp',
    role: 'command',
    badge: 'CIA-01',
    department: 'Operations Command',
    yearsOfService: n/a,
    rank: 'Director'
  }
};
let reports = [];
let cases = [];
let caseCounter = 1001;

// LOCAL STORAGE: Load from localStorage if exists
window.onload = function () {
  const storedUsers = localStorage.getItem('cia_users');
  const storedReports = localStorage.getItem('cia_reports');
  const storedCases = localStorage.getItem('cia_cases');
  const storedCounter = localStorage.getItem('cia_caseCounter');

  if (storedUsers) users = JSON.parse(storedUsers);
  if (storedReports) reports = JSON.parse(storedReports);
  if (storedCases) cases = JSON.parse(storedCases);
  if (storedCounter) caseCounter = parseInt(storedCounter);
};

function saveData() {
  localStorage.setItem('cia_users', JSON.stringify(users));
  localStorage.setItem('cia_reports', JSON.stringify(reports));
  localStorage.setItem('cia_cases', JSON.stringify(cases));
  localStorage.setItem('cia_caseCounter', caseCounter.toString());
}

// AUTO LOGOUT
let logoutTimer;
document.onmousemove = resetTimer;
document.onkeypress = resetTimer;

function resetTimer() {
  clearTimeout(logoutTimer);
  if (currentUser) {
    logoutTimer = setTimeout(() => {
      alert('Session expired. You have been logged out.');
      logout();
    }, 5 * 60 * 1000);
  }
}

// LOGIN LOCKOUT
let failedAttempts = 0;
let lockoutUntil = null;

function login() {
  const now = new Date();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  const message = document.getElementById('loginMessage');

  if (lockoutUntil && now < lockoutUntil) {
    message.innerHTML = '<div class="status-error">Login temporarily locked. Try again later.</div>';
    return;
  }

  if (users[username] && users[username].password === password) {
    currentUser = username;
    failedAttempts = 0;
    showDashboard();
    message.textContent = '';
  } else {
    failedAttempts++;
    if (failedAttempts >= 5) {
      lockoutUntil = new Date(now.getTime() + 10 * 60 * 1000);
      message.innerHTML = '<div class="status-error">Too many failed attempts. Locked out for 10 minutes.</div>';
    } else {
      message.innerHTML = '<div class="status-error">Access Denied. Invalid credentials.</div>';
    }
  }
}

function logout() {
  currentUser = '';
  document.getElementById('dashboard').style.display = 'none';
  document.getElementById('loginPage').style.display = 'flex';
  document.getElementById('username').value = '';
  document.getElementById('password').value = '';
  document.getElementById('loginMessage').innerHTML = '';
}

function showDashboard() {
  document.getElementById('loginPage').style.display = 'none';
  document.getElementById('dashboard').style.display = 'block';
  hideAllSections();
  document.getElementById('dashboardNav').style.display = 'grid';

  const role = users[currentUser].role;
  const userData = users[currentUser];
  document.getElementById('roleTitle').textContent = `${userData.rank} ${currentUser.toUpperCase()}`;
  document.getElementById('badgeInfo').textContent = `Badge: ${userData.badge}`;
  document.getElementById('departmentInfo').textContent = `${userData.department} • ${userData.yearsOfService} years`;
  document.getElementById('userAvatar').textContent = currentUser.charAt(0).toUpperCase();

  // Only supervisors and command see personnel management
  if (role === 'command' || role === 'supervisor') {
    document.getElementById('manageCard').style.display = 'block';
  } else {
    document.getElementById('manageCard').style.display = 'none';
  }
}

function createUser() {
  const newUser = document.getElementById('newUsername').value.trim();
  const newPass = document.getElementById('newPassword').value.trim();
  const newRole = document.getElementById('newRole').value;
  const newDept = document.getElementById('newDepartment').value;
  const newRank = document.getElementById('newRank').value;
  const newYears = document.getElementById('newYears').value || 0;
  const message = document.getElementById('createUserMessage');

  if (newUser && newPass && newRole && newDept && newRank) {
    if (newPass.length < 6) {
      message.innerHTML = '<div class="status-error">Password must be at least 6 characters.</div>';
      return;
    }

    if (!users[newUser]) {
      const badgeNum = Math.floor(Math.random() * 9000) + 1000;
      const badge = `CIA-${badgeNum}`;
      users[newUser] = {
        password: newPass,
        role: newRole,
        badge,
        department: newDept,
        rank: newRank,
        yearsOfService: parseInt(newYears)
      };
      message.innerHTML = `<div class="status-success">Agent '${newUser}' created.<br>Badge: ${badge}</div>`;
      updateAgentsList();
      saveData();
      document.getElementById('newUsername').value = '';
      document.getElementById('newPassword').value = '';
      document.getElementById('newYears').value = '';
    } else {
      message.innerHTML = '<div class="status-warning">Agent already exists.</div>';
    }
  } else {
    message.innerHTML = '<div class="status-error">Please fill in all required fields.</div>';
  }
}

function updateAgentsList() {
  const list = document.getElementById('agentsList');
  list.innerHTML = '';
  const role = users[currentUser].role;

  for (const [name, data] of Object.entries(users)) {
    const card = document.createElement('div');
    card.className = 'agent-card';

    let tools = '';
    if (role === 'command' || role === 'supervisor') {
      tools = `
        <div style="margin-top: 10px;">
          <button onclick="editAgent('${name}')" class="btn-primary" style="padding: 4px 10px; font-size: 10px;">Edit</button>
          <button onclick="deleteAgent('${name}')" class="logout-btn" style="padding: 4px 10px; font-size: 10px;">Delete</button>
        </div>
      `;
    }

    card.innerHTML = `
      <div class="agent-info">
        <div class="agent-name">${data.rank} ${name.toUpperCase()}</div>
        <div class="agent-details">
          <span class="badge-number">${data.badge}</span> • ${data.department}<br>
          Role: ${data.role} • Service: ${data.yearsOfService} years
        </div>
        ${tools}
      </div>
      <div style="text-align: right;">
        <div style="color: #00b894; font-size: 12px; font-weight: 600;">ACTIVE</div>
        <div style="font-size: 10px; color: rgba(255,255,255,0.4);">Online</div>
      </div>
    `;
    list.appendChild(card);
  }
}

function editAgent(name) {
  const data = users[name];
  const newUsername = prompt("Rename agent:", name) || name;
  const newPassword = prompt("Enter new password:", data.password) || data.password;
  const newBadge = prompt("Enter new badge number:", data.badge) || data.badge;

  users[newUsername] = { ...data, password: newPassword, badge: newBadge };
  if (newUsername !== name) delete users[name];

  updateAgentsList();
  saveData();
}

function deleteAgent(name) {
  if (confirm(`Are you sure you want to delete agent '${name}'?`)) {
    delete users[name];
    updateAgentsList();
    saveData();
  }
}

// INTEL SUMMARY
function generateSummary() {
  const input = document.getElementById('reportInput').value.trim();
  const output = document.getElementById('intelSummary');
  if (!input) {
    output.innerHTML = '<div class="status-warning">Enter a report first to generate summary.</div>';
    return;
  }
  const summary = 'Summary: ' + input.split('.').slice(0, 2).join('.') + '...';
  output.innerHTML = `<div><strong>AI Summary:</strong><br>${summary}</div>`;
}
</script>

