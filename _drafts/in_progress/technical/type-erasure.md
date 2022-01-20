watch: Inheritance is the base class of evil

from mechanism to method, valued conversion, C++ Report, kevlin henney

TE is not a void*
not a pointer to base
not a variant

TE is a templated ctor
non-virtual interface
mix of external poly, bridge, proto

we have simple, very basic classes (Circle, Square)

we have a base class (virt dtor), shapeconcept
and as well, abstracts for serialize, draw.

we have a templated model, taking a shape

template<typename GeomShape, typename DrawStrategy>
struct ShapeModel: ShapeConcept {
  ShapeModel(GeomShape const& shape, DrawStrategy strategy) : 
    shape_ (shape), strategy_(strategy) {}
  GeomShape shape_;
  DrawStrategy strategy_;

  void serialize () const override { // forwards to free functions!
    serialize(shape_, /* ... */)
  }

  void draw () const override {
    strategy_(shape_, /* ... */)
  }
};

so far external polymor, with concept and model
  1996 Cleeland

void serialize(Cirtcle const& ...);
void serialize(Square const& ...);

drawAllShapes(vector<unique_ptr<ShapeConcepts>> shapes) {
  for (auto const& shape: shapes)
  {
    shape->draw;
  }
}

int main() {
  using Shapes = vector<unique_ptr<ShapeConcepts>>;
  auto drawRedShape = [color = Color::Red] (auto const& shape, ...) {
    draw(circle, color, /* .. */);
  };
  using DrawStrat = decltype(drawRedShape);

  Shapes shapes;
  shapes.emplace_back(make_unique<ShapeModel<Circle, DrawStrat>>(Circle{...}, drawRed));

  drawAllShapes(shapes);
}

So far, only ext poly, now add more:


class Shape {
  private:
    struct ShapeConcept {
      // ...
      virtual unique_ptr<ShapeConcept> clone() const = 0; // proto DP
    };

    template<typename GeomShape, typename DrawStrategy>
      struct ShapeModel: ShapeConcept {
      ShapeModel(GeomShape const& shape, DrawStrategy strategy) : 
        shape_ (shape), strategy_(strategy) {}
      GeomShape shape_;
      DrawStrategy strategy_;
      // ...

      unique_ptr<ShapeConcept> clone() const override {
        return make_unique<ShapeModel>(*this);
      }

      void serialize () const override { // forwards to free functions!
        serialize(shape_, /* ... */)
      }

      void draw () const override {
        strategy_(shape_, /* ... */)
      }
    };

    friend void draw(Shape const& shape, ...) {
      shape.pimpl->serialize(...);
    }

   unique_ptr<ShapeConcept> pimpl; // the brdige DP, the type is erased! we don't know what's the type we have!

  public:
   template <typename GeomShape, typename DrawStrategy>
    Shape(GeomShape const& shape, DrawStrategy strategy):
    pimp(new ShapeModel<GeomShape, DrawStrategy>(shape, strat)) {}
    //the ctor creates a bridge,takes dependencies togeather

    // default dtor and moves
    // copyies? with clone!

};