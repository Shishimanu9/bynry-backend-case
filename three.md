@app.route('/api/companies/<int:company_id>/alerts/low-stock', methods=['GET'])
def low_stock_alerts(company_id):
    try:
        alerts = []

        # Get warehouses of company
        warehouses = Warehouse.query.filter_by(company_id=company_id).all()

        for wh in warehouses:
            inventories = Inventory.query.filter_by(warehouse_id=wh.id).all()

            for inv in inventories:
                product = Product.query.get(inv.product_id)

                # Check if below threshold
                if inv.quantity < product.threshold:
                    # Get first supplier
                    supplier_rel = ProductSuppliers.query.filter_by(product_id=product.id).first()
                    supplier = Supplier.query.get(supplier_rel.supplier_id) if supplier_rel else None

                    alerts.append({
                        "product_id": product.id,
                        "product_name": product.name,
                        "sku": product.sku,
                        "warehouse_id": wh.id,
                        "warehouse_name": wh.name,
                        "current_stock": inv.quantity,
                        "threshold": product.threshold,
                        "supplier": {
                            "id": supplier.id if supplier else None,
                            "name": supplier.name if supplier else None,
                            "contact_email": supplier.contact_email if supplier else None
                        }
                    })

        return jsonify({
            "alerts": alerts,
            "total_alerts": len(alerts)
        }), 200

    except Exception as e:
        return jsonify({"error": str(e)}), 500
