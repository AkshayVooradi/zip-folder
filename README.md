request from frontend
export const fetchAllFilteredProducts = createAsyncThunk(
  "/products/fetchAllProducts",
  async ({ filterParams, sortParams }) => {
    console.log(fetchAllFilteredProducts, "fetchAllFilteredProducts");

    const query = new URLSearchParams({
      ...filterParams,
      sortBy: sortParams,
    });

    const result = await axios.get(
      `http://localhost:8080/api/product/get?${query}`
    );

    console.log(result);

    return result?.data;
  }
);

how it is to handled in backend node

const getFilteredProducts = async (req, res) => {
  try {
    const { category = [], brand = [], sortBy = "price-lowtohigh" } = req.query;

    let filters = {};

    if (category.length) {
      filters.category = { $in: category.split(",") };
    }

    if (brand.length) {
      filters.brand = { $in: brand.split(",") };
    }

    let sort = {};

    switch (sortBy) {
      case "price-lowtohigh":
        sort.price = 1;

        break;
      case "price-hightolow":
        sort.price = -1;

        break;
      case "title-atoz":
        sort.title = 1;

        break;

      case "title-ztoa":
        sort.title = -1;

        break;

      default:
        sort.price = 1;
        break;
    }

    const products = await Product.find(filters).sort(sort);

    res.status(200).json({
      success: true,
      data: products,
    });
  } catch (e) {
    console.log(error);
    res.status(500).json({
      success: false,
      message: "Some error occured",
    });
  }
};


how i handled it in sprinboot
public ResponseEntity<?> getProducts(List<String> category,List<String> brand,String sortBy) {
        Query query = new Query();

        if(category!=null && !category.isEmpty()){
            query.addCriteria(Criteria.where("category").in(category));
        }

        if(brand!=null && !brand.isEmpty()){
            query.addCriteria(Criteria.where("brand").in(brand));
        }

        Sort sort = Sort.by(Sort.Direction.ASC,"price");

        sort = switch (sortBy) {
            case "price-hightolow" -> Sort.by(Sort.Direction.DESC, "price");
            case "title-atoz" -> Sort.by(Sort.Direction.ASC, "title");
            case "title-ztoa" -> Sort.by(Sort.Direction.DESC, "title");
            default -> sort;
        };

        query.with(sort);

        List<ProductEntity> products = productRepo.find(query);

        Map<String, Object> responseBody = new HashMap<>();
        responseBody.put("success",true);
        responseBody.put("data",products);

        return new ResponseEntity<>(responseBody, HttpStatus.OK);
    }

even extended the interface in my repo
public interface ProductRepo extends MongoRepository<ProductEntity,String>, QuerydslPredicateExecutor<ProductEntity> {

    List<ProductEntity> find(Query query);

    ProductEntity findByTitle(String title);

    List<ProductEntity> findByCategory(String category);

}

and added the dependencies

<dependency>
			<groupId>com.querydsl</groupId>
			<artifactId>querydsl-mongodb</artifactId>
			<version>4.4.0</version>
		</dependency>
		<dependency>
			<groupId>com.querydsl</groupId>
			<artifactId>querydsl-apt</artifactId>
			<version>4.4.0</version>
			<scope>provided</scope>
		</dependency>

but getting the error 
 Error creating bean with name 'publicEPServices': Unsatisfied dependency expressed through field 'productRepo': Error creating bean with name 'productRepo' defined in com.fitnessStore.backend.Repository.ProductRepo defined in @EnableMongoRepositories declared on MongoRepositoriesRegistrar.EnableMongoRepositoriesConfiguration: Did not find a query class com.fitnessStore.backend.Entity.QProductEntity for domain class com.fitnessStore.backend.Entity.ProductEntity

my product entity

package com.fitnessStore.backend.Entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.querydsl.core.annotations.QueryEntity;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.bson.types.ObjectId;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.DBRef;
import org.springframework.data.mongodb.core.mapping.Document;

import java.util.ArrayList;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Document(collection = "product_entity")
@QueryEntity
public class ProductEntity {

    @Id
    private String id;

    @Indexed(unique = true)
    private String title;

    private String category;

    private String brand;

    private double price;

    private double salePrice;

    private String description;

    private String image;

    private int totalStock;

    private double AverageReview= 0;

    private double sumOfRatings = 0;

    @DBRef
    @JsonIgnore
    private List<ReviewEntity> reviews;
}
