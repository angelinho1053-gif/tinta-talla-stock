
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StockApp());

class StockApp extends StatelessWidget {
  const StockApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
    title: 'Tinta & Talla Stock',
    debugShowCheckedModeBanner: false,
    theme: ThemeData.dark().copyWith(
      colorScheme: ColorScheme.fromSeed(
        seedColor: Colors.teal,
        brightness: Brightness.dark,
      ),
    ),
    home: const InventoryPage(),
  );
}

class Product {
  String name;
  String color;
  Map<String, int> sizes;

  Product({
    required this.name,
    required this.color,
    Map<String, int>? sizes,
  }) : sizes = sizes ?? {'CH': 0, 'M': 0, 'G': 0, 'XG': 0};

  int get total => sizes.values.fold(0, (a, b) => a + b);

  Map<String, dynamic> toJson() =>
      {'name': name, 'color': color, 'sizes': sizes};

  factory Product.fromJson(Map<String, dynamic> j) => Product(
    name: j['name'],
    color: j['color'],
    sizes: Map<String, int>.from(j['sizes']),
  );
}

class Movement {
  String product, color, size, type, date;
  int quantity;

  Movement(this.product, this.color, this.size, this.type, this.quantity, this.date);

  Map<String, dynamic> toJson() => {
    'product': product,
    'color': color,
    'size': size,
    'type': type,
    'quantity': quantity,
    'date': date,
  };

  factory Movement.fromJson(Map<String, dynamic> j) => Movement(
    j['product'], j['color'], j['size'], j['type'],
    j['quantity'], j['date'],
  );
}

class InventoryPage extends StatefulWidget {
  const InventoryPage({super.key});

  @override
  State<InventoryPage> createState() => _InventoryPageState();
}

class _InventoryPageState extends State<InventoryPage> {
  List<Product> products = [];
  List<Movement> movements = [];
  bool loading = true;
  String query = '';

  @override
  void initState() {
    super.initState();
    load();
  }

  Future<void> load() async {
    final prefs = await SharedPreferences.getInstance();
    final raw = prefs.getString('products');
    final hist = prefs.getString('movements');

    if (raw != null) {
      products = (jsonDecode(raw) as List)
          .map((e) => Product.fromJson(Map<String, dynamic>.from(e)))
          .toList();
    }

    if (hist != null) {
      movements = (jsonDecode(hist) as List)
          .map((e) => Movement.fromJson(Map<String, dynamic>.from(e)))
          .toList();
    }

    if (products.isEmpty) {
      products.add(Product(
        name: 'Playera Algodón - Diseño Rock',
        color: 'Negro',
        sizes: {'CH': 4, 'M': 8, 'G': 6, 'XG': 3},
      ));
      await save();
    }

    setState(() => loading = false);
  }

  Future<void> save() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(
      'products',
      jsonEncode(products.map((e) => e.toJson()).toList(),
    );
    await prefs.setString(
      'movements',
      jsonEncode(movements.map((e) => e.toJson()).toList(),
    );
  }

  int get totalStock => products.fold(0, (a, p) => a + p.total);

  List<Product> get filtered => products.where((p) =>
      '${p.name} ${p.color}".toLowerCase().contains(query.toLowerCase())).toList();

  void snack(String text) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(text)),
    );
  }

  Future<void> addProduct() async {
    final name = TextEditingController();
    final color = TextEditingController();

    final ok = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Nuevo modelo'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: name, decoration: const InputDecoration(labelText: 'Modelo')),
            TextField(controller: color, decoration: const InputDecoration(labelText: 'Color')),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context, false),
              child: const Text('Cancelar')),
          FilledButton(onPressed: () => Navigator.pop(context, true),
              child: const Text('Guardar')),
        ],
      ),
    );

    if (ok == true && name.text.trim().isNotEmpty &&
        color.text.trim().isNotEmpty) {
      setState(() => products.add(Product(
        name: name.text.trim(), color: color.text.trim(),
      )));
      await save();
    }
  }

  Future<void> movement(Product p, String size, String type) async {
    final controller = TextEditingController(text: '1');

    final ok = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: Text(type),
        content: TextField(
          controller: controller,
          keyboardType: TextInputType.number,
          decoration: const InputDecoration(labelText: 'Cantidad'),
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context, false),
              child: const Text('Cancelar')),
          FilledButton(onPressed: () => Navigator.pop(context, true),
              child: const Text('Confirmar')),
        ],
      ),
    );

    if (ok != true) return;

    final qty = int.tryParse(controller.text) ?? 0;
    if (qty <= 0) {
      snack('Cantidad inválida');
      return;
    }

    if (type == 'Salida' && p.sizes[size]! < qty) {
      snack('Stock insuficiente');
      return;
    }

    setState(() {
      p.sizes[size] = p.sizes[size]! + (type == 'Entrada' ? qty : -qty);
      movements.insert(0, Movement(
        p.name, p.color, size, type, qty,
        DateTime.now().toString().substring(0, 16),
      ));
    });

    await save();
    snack('$type registrada');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Tinta & Talla Stock'),
        actions: [
          IconButton(
            icon: const Icon(Icons.history),
            onPressed: () => Navigator.push(context,
              MaterialPageRoute(builder: (_) => HistoryPage(items: movements))),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: addProduct,
        icon: const Icon(Icons.add),
        label: const Text('Nuevo modelo'),
      ),
      body: loading
          ? const Center(child: CircularProgressIndicator())
          : Column(children: [
              Card(
                margin: const EdgeInsets.all(12),
                child: ListTile(
                  leading: const Icon(Icons.inventory, size: 35),
                  title: const Text('Inventario total'),
                  subtitle: Text('$totalStock piezas',
                      style: Theme.of(context).textTheme.headlineMedium),
                ),
              ),
              Padding(
                padding: const EdgeInsets.symmetric(horizontal: 12),
                child: TextField(
                  onChanged: (v) => setState(() => query = v),
                  decoration: const InputDecoration(
                    prefixIcon: Icon(Icons.search),
                    hintText: 'Buscar modelo o color',
                  ),
                ),
              ),
              Expanded(
                child: ListView.builder(
                  padding: const EdgeInsets.all(12),
                  itemCount: filtered.length,
                  itemBuilder: (_, i) {
                    final p = filtered[i];
                    return Card(
                      child: Padding(
                        padding: const EdgeInsets.all(12),
                        child: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            Text(p.name, style: Theme.of(context).textTheme.titleLarge),
                            Text('Color: ${p.color} • Total: ${p.total}'),
                            const SizedBox(height: 10),
                            Wrap(
                              spacing: 8,
                              children: p.sizes.keys.map((s) => SizedBox(
                                width: 75,
                                child: Column(children: [
                                  Text(s),
                                  Text('${p.sizes[s]} piezas'),
                                  Row(children: [
                                    IconButton(
                                      icon: const Icon(Icons.remove_circle),
                                      onPressed: () => movement(p, s, 'Salida'),
                                    ),
                                    IconButton(
                                      icon: const Icon(Icons.add_circle),
                                      onPressed: () => movement(p, s, 'Entrada'),
                                    ),
                                  ]),
                                ]),
                              )).toList(),
                            ),
                          ],
                        ),
                      ),
                    );
                  },
                ),
              ),
            ]),
    );
  }
}

class HistoryPage extends StatelessWidget {
  final List<Movement> items;
  const HistoryPage({super.key, required this.items});

  @override
  Widget build(BuildContext context) => Scaffold(
    appBar: AppBar(title: const Text('Historial')),
    body: items.isEmpty
        ? const Center(child: Text('Sin movimientos'))
        : ListView.builder(
            itemCount: items.length,
            itemBuilder: (_, i) {
              final m = items[i];
              final entry = m.type == 'Entrada';
              return ListTile(
                leading: Icon(entry ? Icons.add_circle : Icons.remove_circle),
                title: Text('${m.type}: ${m.quantity} × ${m.size}'),
                subtitle: Text('${m.product} • ${m.color}\n${m.date}'),
                isThreeLine: true,
              );
            },
          ),
  );
}
