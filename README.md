import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: DoseCalculatorScreen(),
    );
  }
}

class DoseCalculatorScreen extends StatefulWidget {
  const DoseCalculatorScreen({super.key});

  @override
  State<DoseCalculatorScreen> createState() => _DoseCalculatorScreenState();
}

class _DoseCalculatorScreenState extends State<DoseCalculatorScreen> {
  final TextEditingController _inrController = TextEditingController();
  final TextEditingController _currentDoseController = TextEditingController();
  String _doseRecommendation = '';
  List<String> _doseHistory = [];

  @override
  void initState() {
    super.initState();
    _loadDoseHistory();
  }

  Future<void> _loadDoseHistory() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _doseHistory = prefs.getStringList('doseHistory') ?? [];
    });
  }

  Future<void> _saveDoseHistory() async {
    final prefs = await SharedPreferences.getInstance();
    prefs.setStringList('doseHistory', _doseHistory);
  }

  void _calculateDose() {
    double? inr = double.tryParse(_inrController.text.replaceAll(',', '.'));
    double? currentDose = double.tryParse(
      _currentDoseController.text.replaceAll(',', '.'),
    );

    if (inr == null || inr <= 0 || currentDose == null || currentDose <= 0) {
      setState(() {
        _doseRecommendation = 'أدخل INR وجرعة صحيحة';
      });
      return;
    }

    String newDose;
    if (inr < 2.0) {
      newDose = (currentDose * 1.2).toStringAsFixed(1);
      setState(() {
        _doseRecommendation = 'قم بزيادة الجرعة إلى $newDose ملغ';
      });
    } else if (inr > 3.0) {
      newDose = (currentDose * 0.8).toStringAsFixed(1);
      setState(() {
        _doseRecommendation = 'قم بتقليل الجرعة إلى $newDose ملغ';
      });
    } else {
      setState(() {
        _doseRecommendation = 'الجرعة الحالية مناسبة';
      });
    }

    _doseHistory.add('INR: $inr, الجرعة الموصى بها: $_doseRecommendation');
    _saveDoseHistory();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('حساب جرعة الوارفارين')),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            TextField(
              controller: _inrController,
              keyboardType: const TextInputType.numberWithOptions(
                decimal: true,
              ),
              decoration: const InputDecoration(
                labelText: 'أدخل قيمة INR',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 20),
            TextField(
              controller: _currentDoseController,
              keyboardType: const TextInputType.numberWithOptions(
