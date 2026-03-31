const express = require("express");
const cors = require("cors");
const { v4: uuid } = require("uuid");

const app = express();

// ✅ Middleware
app.use(cors());
app.use(express.json());

// ✅ Use Render PORT
const PORT = process.env.PORT || 3000;

// ✅ In-memory DB (temporary)
let groups = [];

// 🔥 Helper → normalize names
const normalize = (name) => name.trim().toLowerCase();

// ---------- HEALTH CHECK (IMPORTANT for Render) ----------
app.get("/", (req, res) => {
  res.send("Backend is running 🚀");
});

// ---------- CREATE GROUP ----------
app.post("/groups", (req, res) => {
  const { name, members } = req.body;

  if (!name || !Array.isArray(members)) {
    return res.status(400).json({ error: "Invalid data" });
  }

  const cleanMembers = members
    .map(m => m.trim())
    .filter(m => m !== "");

  if (cleanMembers.length === 0) {
    return res.status(400).json({ error: "Members required" });
  }

  const uniqueMembers = [...new Set(cleanMembers)];

  const group = {
    id: uuid(),
    name: name.trim(),
    members: uniqueMembers,
    expenses: []
  };

  groups.push(group);
  res.json(group);
});

// ---------- GET GROUPS ----------
app.get("/groups", (req, res) => {
  res.json(groups);
});

// ---------- DELETE GROUP ----------
app.delete("/groups/:id", (req, res) => {
  groups = groups.filter(g => g.id !== req.params.id);
  res.json({ message: "Group deleted" });
});

// ---------- ADD EXPENSE ----------
app.post("/groups/:id/expense", (req, res) => {
  const group = groups.find(g => g.id === req.params.id);
  if (!group) return res.status(404).json({ error: "Group not found" });

  let { desc, amount, paidBy } = req.body;

  if (!desc || !amount || !paidBy) {
    return res.status(400).json({ error: "Missing fields" });
  }

  amount = Number(amount);
  if (isNaN(amount) || amount <= 0) {
    return res.status(400).json({ error: "Invalid amount" });
  }

  const payer = paidBy.trim();

  const isValidUser = group.members.some(
    m => normalize(m) === normalize(payer)
  );

  if (!isValidUser) {
    return res.status(400).json({ error: "User not in group" });
  }

  const expense = {
    id: uuid(),
    desc: desc.trim(),
    amount,
    paidBy: payer,
    date: new Date().toISOString()
  };

  group.expenses.push(expense);
  res.json(expense);
});

// ---------- EDIT EXPENSE ----------
app.put("/groups/:id/expense/:index", (req, res) => {
  const group = groups.find(g => g.id === req.params.id);
  if (!group) return res.status(404).json({ error: "Group not found" });

  const index = parseInt(req.params.index);
  if (isNaN(index) || index < 0 || index >= group.expenses.length) {
    return res.status(400).json({ error: "Invalid index" });
  }

  let { desc, amount, paidBy } = req.body;

  if (!desc || !amount || !paidBy) {
    return res.status(400).json({ error: "Missing fields" });
  }

  amount = Number(amount);
  if (isNaN(amount) || amount <= 0) {
    return res.status(400).json({ error: "Invalid amount" });
  }

  const payer = paidBy.trim();

  const isValidUser = group.members.some(
    m => normalize(m) === normalize(payer)
  );

  if (!isValidUser) {
    return res.status(400).json({ error: "User not in group" });
  }

  group.expenses[index] = {
    ...group.expenses[index],
    desc: desc.trim(),
    amount,
    paidBy: payer,
    date: new Date().toISOString()
  };

  res.json(group.expenses[index]);
});

// ---------- DELETE EXPENSE ----------
app.delete("/groups/:id/expense/:index", (req, res) => {
  const group = groups.find(g => g.id === req.params.id);
  if (!group) return res.status(404).json({ error: "Group not found" });

  const index = parseInt(req.params.index);
  if (isNaN(index) || index < 0 || index >= group.expenses.length) {
    return res.status(400).json({ error: "Invalid index" });
  }

  group.expenses.splice(index, 1);
  res.json({ message: "Expense deleted" });
});

// ---------- UPDATE MEMBERS ----------
app.put("/groups/:id/members", (req, res) => {
  const group = groups.find(g => g.id === req.params.id);
  if (!group) return res.status(404).json({ error: "Group not found" });

  if (!Array.isArray(req.body.members)) {
    return res.status(400).json({ error: "Invalid members list" });
  }

  const cleanMembers = req.body.members
    .map(m => m.trim())
    .filter(m => m !== "");

  if (cleanMembers.length === 0) {
    return res.status(400).json({ error: "Members cannot be empty" });
  }

  const uniqueMembers = [...new Set(cleanMembers)];

  // ❌ prevent removing users used in expenses
  const invalidExpense = group.expenses.find(e =>
    !uniqueMembers.some(m => normalize(m) === normalize(e.paidBy))
  );

  if (invalidExpense) {
    return res.status(400).json({
      error: `Cannot remove "${invalidExpense.paidBy}" — used in expenses`
    });
  }

  group.members = uniqueMembers;

  res.json(group);
});

// ---------- CALCULATE BALANCE ----------
app.get("/groups/:id/balance", (req, res) => {
  const group = groups.find(g => g.id === req.params.id);
  if (!group) return res.status(404).json({ error: "Group not found" });

  let balance = {};

  group.members.forEach(m => balance[m] = 0);

  group.expenses.forEach(e => {
    const share = e.amount / group.members.length;

    group.members.forEach(m => {
      balance[m] -= share;
    });

    const payerKey = group.members.find(
      m => normalize(m) === normalize(e.paidBy)
    );

    if (payerKey) {
      balance[payerKey] += e.amount;
    }
  });

  res.json(balance);
});

// ---------- START SERVER ----------
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
