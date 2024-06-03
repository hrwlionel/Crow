#include "crow_all.h"
#include <fstream>
#include <sstream>
#include <vector>
#include <string>

struct ArtPiece {
    std::string title;
    std::string description;
    std::string image_url;
    double price;
};

// In-memory storage for art pieces (for simplicity)
std::vector<ArtPiece> artPieces;

void loadArtPieces() {
    // For demonstration purposes, load some art pieces
    artPieces.push_back({"Sunset", "A beautiful sunset", "/static/sunset.jpg", 100.0});
    artPieces.push_back({"Mountain", "A majestic mountain", "/static/mountain.jpg", 200.0});
}

int main() {
    crow::SimpleApp app;

    loadArtPieces();

    // Route to serve static files
    CROW_ROUTE(app, "/static/<string>")
    ([](crow::response& res, const std::string& filename){
        std::ifstream file("static/" + filename, std::ios::binary);
        if (file) {
            std::ostringstream contents;
            contents << file.rdbuf();
            res.write(contents.str());
            res.end();
        } else {
            res.code = 404;
            res.write("File not found");
            res.end();
        }
    });

    // Route to list all art pieces
    CROW_ROUTE(app, "/")
    ([](){
        crow::mustache::context ctx;
        ctx["artPieces"] = crow::json::load(R"(
            [{"title":"Sunset","description":"A beautiful sunset","image_url":"/static/sunset.jpg","price":100.0},
             {"title":"Mountain","description":"A majestic mountain","image_url":"/static/mountain.jpg","price":200.0}]
        )");
        auto page = crow::mustache::load("views/index.html");
        return page.render(ctx);
    });

    // Route to buy art piece (simplified for demonstration)
    CROW_ROUTE(app, "/buy/<int>")
    ([](int id){
        if (id >= 0 && id < artPieces.size()) {
            return crow::response(200, "Thank you for buying " + artPieces[id].title);
        } else {
            return crow::response(404, "Art piece not found");
        }
    });

    // Set the views directory
    crow::mustache::set_base(".");

    app.port(8080).multithreaded().run();

    return 0;
}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Art Gallery</title>
    <style>
        .art-piece {
            border: 1px solid #ccc;
            padding: 10px;
            margin: 10px;
        }
        .art-piece img {
            max-width: 200px;
            height: auto;
        }
    </style>
</head>
<body>
    <h1>Art Gallery</h1>
    <div id="artGallery">
        {{#artPieces}}
        <div class="art-piece">
            <h2>{{title}}</h2>
            <p>{{description}}</p>
            <img src="{{image_url}}" alt="{{title}}">
            <p>Price: ${{price}}</p>
            <a href="/buy/{{@index}}">Buy</a>
        </div>
        {{/artPieces}}
    </div>
</body>
</html>
g++ main.cpp -o artgallery -std=c++11 -I/path/to/boost -I/path/to/crow/include -L/path/to/boost/lib -lboost_system
./artgallery


