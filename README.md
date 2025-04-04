request from frontend for add to cart
export const addToCart = createAsyncThunk(
  "cart/addToCart",
  async ({ userId, productId, quantity }) => {
    const response = await axios.post(
      "http://localhost:8080/api/cart/add",
      {
        productId,
        quantity,
      },{
        withCredentials: true
      }
    );
    return response.data;
  }
);

node js implementation for add to cart
const addToCart = async (req, res) => {
  try {
    const { userId, productId, quantity } = req.body;

    if (!userId || !productId || quantity <= 0) {
      return res.status(400).json({
        success: false,
        message: "Invalid data provided!",
      });
    }

    const product = await Product.findById(productId);

    if (!product) {
      return res.status(404).json({
        success: false,
        message: "Product not found",
      });
    }

    let cart = await Cart.findOne({ userId });

    if (!cart) {
      cart = new Cart({ userId, items: [] });
    }

    const findCurrentProductIndex = cart.items.findIndex(
      (item) => item.productId.toString() === productId
    );

    if (findCurrentProductIndex === -1) {
      cart.items.push({ productId, quantity });
    } else {
      cart.items[findCurrentProductIndex].quantity += quantity;
    }

    await cart.save();
    res.status(200).json({
      success: true,
      data: cart,
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({
      success: false,
      message: "Error",
    });
  }
};

my implementation of add to cart using spring boot
@PostMapping("/add")
    public ResponseEntity<?> AddProduct(@CookieValue(value = "token",defaultValue = "")String token, Map<String ,String> credentials){
        return cartServices.addProduct(credentials.get("productId"),credentials.get("quantity"),token);
    }

    public ResponseEntity<?> addProduct(String productId,String quantity,String authHeader) {
        UserEntity user = userByToken.userDetails(authHeader);

        Map<String,Object> responseBody = new HashMap<>();

        if(productId.isEmpty() || quantity.isEmpty()){
            responseBody.put("success",false);
            responseBody.put("message","Invalid data provided!");
            return new ResponseEntity<>(responseBody,HttpStatus.BAD_REQUEST);
        }

        CartEntity cart = cartRepo.findByUser(user);

        if(cart == null){
            cart = CartEntity.builder()
                    .user(user)
                    .products(new ArrayList<>())
                    .build();
            cartRepo.save(cart);
            user.setCart(cart);
            userRepo.save(user);
        }

        Optional<ProductEntity> product = productRepo.findById(productId);

        if(product.isEmpty()){
            responseBody.put("success",false);
            responseBody.put("message","Product not found");
            return new ResponseEntity<>(responseBody,HttpStatus.NOT_FOUND);
        }

        List<CartItemClass> products = cart.getProducts();

        boolean present=false;

        CartItemClass newItem = null;

        //if product is already present inside the cart
        for(CartItemClass prod: products){
            if(prod.getProduct().getId().equals(product.get().getId())){
                prod.setQuantity(prod.getQuantity()+Integer.parseInt(quantity));
                present=true;
                newItem=prod;
                break;
            }
        }

        if(!present) {
            CartItemClass item = CartItemClass.builder()
                    .product(product.get())
                    .productId(productId)
                    .quantity(Integer.parseInt(quantity))
                    .title(product.get().getTitle())
                    .image(product.get().getImage())
                    .price(product.get().getPrice())
                    .salePrice(product.get().getSalePrice())
                    .build();
            cart.getProducts().add(item);
            newItem=item;
        }

        cartRepo.save(cart);

        responseBody.put("success",true);
        responseBody.put("data",cart);

        return new ResponseEntity<>(responseBody, HttpStatus.OK);
    }

    my cart entity
    
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Document(collection = "cart_entity")
@EqualsAndHashCode(exclude = "user")
public class CartEntity {

    @Id
    private ObjectId id;

    @DBRef
    @Field("user_id")
    @JsonIgnore
    private UserEntity user;

    private List<CartItemClass> products;
}

my cart item class
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Data
@Component
public class CartItemClass {

    @DBRef
    private ProductEntity product;

    private String productId;

    private int quantity;

    private String size;

    private String title;

    private String image;

    private double price;

    private double salePrice;

    private String status;

    private LocalDateTime deliveredAt;
}
 i am getting user from cookie token where as node js code was sending it again
