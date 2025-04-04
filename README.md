my request from frontend to fetch all products 
export const fetchCartItems = createAsyncThunk(
  "cart/fetchCartItems",
  async (userId) => {
    const response = await axios.get(
      `http://localhost:8080/api/cart/get`,{
        withCredentials:true
      }
    );

    return response.data;
  }
);

my node js impelmentation

const fetchCartItems = async (req, res) => {
  try {
    const { userId } = req.params;

    if (!userId) {
      return res.status(400).json({
        success: false,
        message: "User id is manadatory!",
      });
    }

    const cart = await Cart.findOne({ userId }).populate({
      path: "items.productId",
      select: "image title price salePrice",
    });

    if (!cart) {
      return res.status(404).json({
        success: false,
        message: "Cart not found!",
      });
    }

    const validItems = cart.items.filter(
      (productItem) => productItem.productId
    );

    if (validItems.length < cart.items.length) {
      cart.items = validItems;
      await cart.save();
    }

    const populateCartItems = validItems.map((item) => ({
      productId: item.productId._id,
      image: item.productId.image,
      title: item.productId.title,
      price: item.productId.price,
      salePrice: item.productId.salePrice,
      quantity: item.quantity,
    }));

    res.status(200).json({
      success: true,
      data: {
        ...cart._doc,
        items: populateCartItems,
      },
    });
  } catch (error) {
    console.log(error);
    res.status(500).json({
      success: false,
      message: "Error",
    });
  }
};

mysprinboot implementation
public ResponseEntity<?> getProducts(String authHeader) {
        UserEntity user = userByToken.userDetails(authHeader);

        CartEntity cart = user.getCart();

        Map<String,Object> responseBody = new HashMap<>();

        if(cart == null){
            responseBody.put("success",false);
            responseBody.put("message","Cart not found");
            return new ResponseEntity<>(responseBody,HttpStatus.NOT_FOUND);
        }

        responseBody.put("success",false);
        responseBody.put("data")

        return new ResponseEntity<>(cart.getProducts(),HttpStatus.OK);
    }

    i am having doubt on how to implement similar functionality as node
