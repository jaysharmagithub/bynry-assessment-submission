
Part 1
I check data is not blank and check each attribute is not blank and check price are in correct decimal and then check if product exit by sku earlier if yes then alert the client by throwing excpetion need to define the exception if not defined and throw it
similarly do for warehoues id as well -
get it product id saved in database  and ensure that product object isn't null and not blank 
check the inventory exit or not for that product else throw Exceptin (custom ) 
Again check for warehouse as well that it exist or not in database or else throw exceptin  
create the inventory product and update the inventory and return resposne isn't structure and no code and no time stamp no path and no data object  and  api isn't followed acid which give not inconsistnet resunlt and all above is not present in code provided

Part 2
		   Warehouse
			|
Supplier - product - company - warehouse 
                      |
		  bundle 

C one to many W -  company will have foreign kwy of wareshouse not null, id not null unqiue . name not null and upto 255 char and  warehouse table - id not null, unqiue, name not null upto 255, 
P one to many Warehouse - P table will have foregin key of ware house which not null, id not null and unqiue and warehouse follow the same of above 
S many to many P - Product can have Multiple suppier and Supplier can have multi product thus will create seperate table for s_id and p_id both null and each unique
P many to many C - Product can be assoicated many companies and Company can have multi product thus will create seperate table for c_id and p_id both null and each unique
Table Inventory with Product_id not null unique, Ware-house_id not null, quantity


Part 3
Database Design
Product[id(not null, unqiue,primary_key),name,sku(not null, unqiue), threshold(not null),createdAt not null, createdBy not null, updatedAt not null, updatedBy not null]
Inventory[id(not null, unqiue,primary_key),product_id(not null, foregin key), warehoue_id(not null), quantity(not null), createdAt(not null), updateAt(not null), createdBy(not null), updatedBy(not null)]
Warehous[id(not null, unqiue,primary_key),product_id(not null, foregin key), company_id(not null), createdAt(not null), updateAt(not null), createdBy(not null), updatedBy(not null)]

first check that company exit or not by id if not throw custom defined exception
get the products associated with the company and get threshold  and get their quantity for 30 days(assumped for one month) from Inventory  and  check if threshold> stock  and send alert for that product 

@Getter
@Setter
@NoArgsConstructor
@ReqArgsConstructor
public class LowStockAlertDTO { private Long productId; private String productName; private String sku; private Long warehouseId; private String warehouseName; private int currentStock; private int threshold; private Integer daysUntilStockout; private SupplierDTO supplier;}

@Getter
@Setter
@NoArgsConstructor
@ReqArgsConstructor
public class SupplierDTO { private Long id; private String name; private String contactEmail;
}
private final ProductRepository productRepo;
private final InventoryRepository inventoryRepo;
private final InventoryChangeRepository changeRepo;
private final SupplierProductRepository supplierProductRepo;

public LowStockAlertService(ProductRepository productRepo,
                            InventoryRepository inventoryRepo,
                            InventoryChangeRepository changeRepo,
                            SupplierProductRepository supplierProductRepo) {
    this.productRepo = productRepo;
    this.inventoryRepo = inventoryRepo;
    this.changeRepo = changeRepo;
    this.supplierProductRepo = supplierProductRepo;
}

public List<LowStockAlertDTO> getLowStockAlerts(Long companyId) {
Company c = findById(companyId).orElseThrow(() -> new CompanyNotFoundException("The company not found");
    List<LowStockAlertDTO> alerts = new ArrayList<>();
    List<Product> products = productRepo.findByCompanyId(companyId);
    LocalDateTime thirtyDays = LocalDateTime.now().minusDays(30);
    for (Product product : products) {
        Integer threshold = product.getThreshold();
        List<Inventory> inventories = inventoryRepo.findByProductId(product.getId());
        for (Inventory inventory : inventories) {
            int currentStock = inventory.getStockOfPast(thirtyDays);
            int avgDailySales = currentStock / 30;
            Integer daysUntilStockout = (avgDailySales > 0) ? (currentStock / avgDailySales) : null;
            SupplierDTO supplierDTO = null;
            supplierProductRepo.findTopByProductId(product.getId()).ifPresent(sp -> {
              Supplier supplier = sp.getSupplier();
                supplierDTO = new SupplierDTO(supplier.getId(), supplier.getName(), supplier.getContactDetails());
            });
            alerts.add(new LowStockAlertDTO(
                    product.getId(),
                    product.getProductName(),
                    product.getSku(),
                    inventory.getWarehouse().getId(),
                    inventory.getWarehouse().getName(),
                    currentStock,
                    threshold,
                    daysUntilStockout,
                    supplierDTO
            ));
        }
    }
    return alerts;
}
        
