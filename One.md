PART-1- Debugging and Code Review
1.SKU Uniquness- 
       *Code directly saves product without checking uniquneness of SKU. If this
       multiple same SKU would be stored.
       *Solution- We can add a contrain in database to check uniqueness
2. Single Warehouse- 
          *A product has warehouse id but a single product can exist in
            multiple warehouses
          *solution- remove wareshouse id and let ineventory handle it
3.Price in float- 
           *Price must in decimal or float will cause rounding errors
           *solution- use decimal instead of float
4.Input validation-
           *User can not always provide all required information
           *solution-validate input or check input before creating product
5.Multiple commits- 
            *Risk to atomicity
            *solution- use one commit to save all
6.Error Handling-
             *errors can occur in db
             *solution- use of try and catch method
7.Creation of inventory-
             *for every product creation inventory get created
             *solution- make it optional through some attributes
           
#corrected code
from decimal import Decimal
from flask import request, jsonify
from sqlalchemy.exc import IntegrityError
from app import db
from models import Product, Inventory

@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json

    try:
        
        if 'name' not in data or 'sku' not in data or 'price' not in data:
            return jsonify({"error": "Missing required fields"}), 400

      
        existing = Product.query.filter_by(sku=data['sku']).first()
        if existing:
            return jsonify({"error": "SKU already exists"}), 400

        
        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=Decimal(str(data['price']))
        )
        db.session.add(product)
        db.session.flush()  # get product.id before commit

       
        if 'warehouse_id' in data and 'initial_quantity' in data:
            inventory = Inventory(
                product_id=product.id,
                warehouse_id=data['warehouse_id'],
                quantity=data['initial_quantity']
            )
            db.session.add(inventory)

        
        db.session.commit()

        return jsonify({"message": "Product created", "product_id": product.id}), 201

    except IntegrityError:
        db.session.rollback()
        return jsonify({"error": "Database integrity error"}), 500
    except Exception as e:
        db.session.rollback()
        return jsonify({"error": str(e)}), 500
