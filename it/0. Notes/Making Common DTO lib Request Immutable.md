To make the `RatingRequest` class immutable, you must ensure the following:

1. All fields are `final`, so their values can only be set once.
2. Remove setters or any method that allows modification after construction.
3. Provide a constructor to initialize all fields during object creation.
4. Use defensive copies if any field is mutable (e.g., `Rating` or `String` in this case).
5. Ensure the `Consumer`-based method works with the immutable class by returning a new instance rather than modifying an existing one.

Here's how your class can be refactored:

```java
import lombok.ToString;
import java.util.function.Consumer;

@ToString
public final class RatingRequest {

    private final Integer userId;
    private final Integer productId;
    private final Rating rating;
    private final String comment;

    private RatingRequest(Integer userId, Integer productId, Rating rating, String comment) {
        this.userId = userId;
        this.productId = productId;
        this.rating = rating; // Consider defensive copying if Rating is mutable
        this.comment = comment; // Strings are immutable, so no need for copying
    }

    public Integer getUserId() {
        return userId;
    }

    public Integer getProductId() {
        return productId;
    }

    public Rating getRating() {
        return rating; // Consider returning a defensive copy if Rating is mutable
    }

    public String getComment() {
        return comment;
    }

    public static RatingRequest newRequest(Consumer<RatingRequestBuilder> consumer) {
        RatingRequestBuilder builder = new RatingRequestBuilder();
        consumer.accept(builder);
        return builder.build();
    }

    public static class RatingRequestBuilder {
        private Integer userId;
        private Integer productId;
        private Rating rating;
        private String comment;

        public RatingRequestBuilder setUserId(Integer userId) {
            this.userId = userId;
            return this;
        }

        public RatingRequestBuilder setProductId(Integer productId) {
            this.productId = productId;
            return this;
        }

        public RatingRequestBuilder setRating(Rating rating) {
            this.rating = rating; // Consider defensive copying if Rating is mutable
            return this;
        }

        public RatingRequestBuilder setComment(String comment) {
            this.comment = comment;
            return this;
        }

        private RatingRequest build() {
            return new RatingRequest(userId, productId, rating, comment);
        }
    }
}
```

### Key Changes:

1. The `RatingRequest` class is marked as `final` to prevent subclassing.
2. Fields are marked `final` and initialized via a private constructor.
3. A builder class `RatingRequestBuilder` is introduced to support the construction process, allowing the `Consumer` pattern to work seamlessly.
4. Defensive copying should be considered for the `Rating` field if it is mutable.