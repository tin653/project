import 'package:flutter/material.dart';

// --- Data Model ---
class Expense {
  String id;
  String title;
  double amount;

  Expense({required this.id, required this.title, required this.amount});
}

void main() => runApp(const MaterialApp(home: ExpenseListScreen()));

// --- Screen 1: Expense List ---
class ExpenseListScreen extends StatefulWidget {
  const ExpenseListScreen({super.key});

  @override
  State<ExpenseListScreen> createState() => _ExpenseListScreenState();
}

class _ExpenseListScreenState extends State<ExpenseListScreen> {
  final List<Expense> _expenses = [
    Expense(id: '1', title: 'Dog Food', amount: 12.50),
    Expense(id: '2', title: 'Gas', amount: 45.00),
    Expense(id: '3', title: 'Internet Bill', amount: 30.00),
  ];

  // Logic to navigate and handle results (Part 2 & 3)
  Future<void> _navigateToForm([Expense? expense]) async {
    final result = await Navigator.push<Expense>(
      context,
      MaterialPageRoute(
        builder: (context) => ExpenseFormScreen(expense: expense),
      ),
    );

    if (result == null || !mounted) return;

    setState(() {
      if (expense == null) {
        _expenses.add(result); // Add new
      } else {
        final index = _expenses.indexWhere((e) => e.id == expense.id);
        _expenses[index] = result; // Update existing
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Expense Tracker')),
      body: _expenses.isEmpty
          ? const Center(child: Text('No expenses yet.'))
          : ListView.builder(
              itemCount: _expenses.length,
              itemBuilder: (context, index) {
                final item = _expenses[index];
                return Card(
                  margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                  child: ListTile(
                    title: Text(item.title),
                    trailing: Text('\$${item.amount.toStringAsFixed(2)}'),
                    onTap: () => _navigateToForm(item), // Trigger Edit
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _navigateToForm(), // Trigger Add
        child: const Icon(Icons.add),
      ),
    );
  }
}

// --- Screen 2: Add/Edit Form (Part 2 & 3) ---
class ExpenseFormScreen extends StatefulWidget {
  final Expense? expense; // If null, we are in "Add" mode

  const ExpenseFormScreen({super.key, this.expense});

  @override
  State<ExpenseFormScreen> createState() => _ExpenseFormScreenState();
}

class _ExpenseFormScreenState extends State<ExpenseFormScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _titleController;
  late TextEditingController _amountController;

  @override
  void initState() {
    super.initState();
    // Prefill if editing
    _titleController = TextEditingController(text: widget.expense?.title ?? '');
    _amountController = TextEditingController(
      text: widget.expense != null ? widget.expense!.amount.toString() : '',
    );
  }

  void _save() {
    if (_formKey.currentState!.validate()) {
      final newExpense = Expense(
        id: widget.expense?.id ?? DateTime.now().toString(),
        title: _titleController.text,
        amount: double.parse(_amountController.text),
      );
      Navigator.pop(context, newExpense); // Return result
    }
  }

  @override
  Widget build(BuildContext context) {
    final isEditing = widget.expense != null;

    return Scaffold(
      appBar: AppBar(title: Text(isEditing ? 'Edit Expense' : 'Add Expense')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              TextFormField(
                controller: _titleController,
                decoration: const InputDecoration(labelText: 'Title'),
                validator: (val) => (val == null || val.isEmpty) ? 'Enter a title' : null,
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _amountController,
                decoration: const InputDecoration(labelText: 'Amount'),
                keyboardType: TextInputType.number,
                validator: (val) {
                  if (val == null || double.tryParse(val) == null || double.parse(val) <= 0) {
                    return 'Enter a valid amount > 0';
                  }
                  return null;
                },
              ),
              const Spacer(),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancel')),
                  ElevatedButton(onPressed: _save, child: const Text('Save Expense')),
                ],
              )
            ],
          ),
        ),
      ),
    );
  }
}
