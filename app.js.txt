// app.js
(function () {
    'use strict';

    angular.module('NarrowItDownApp', [])
    .controller('NarrowItDownController', NarrowItDownController)
    .service('MenuSearchService', MenuSearchService)
    .directive('foundItems', FoundItemsDirective);

    NarrowItDownController.$inject = ['MenuSearchService'];
    function NarrowItDownController(MenuSearchService) {
        var ctrl = this;
        ctrl.searchTerm = "";
        ctrl.found = [];
        ctrl.nothingFound = false;

        ctrl.narrowItDown = function () {
            if (ctrl.searchTerm.trim() === "") {
                ctrl.found = [];
                ctrl.nothingFound = true;
                return;
            }

            MenuSearchService.getMatchedMenuItems(ctrl.searchTerm)
                .then(function (response) {
                    ctrl.found = response;
                    ctrl.nothingFound = ctrl.found.length === 0;
                });
        };

        ctrl.removeItem = function (index) {
            ctrl.found.splice(index, 1);
        };
    }

    MenuSearchService.$inject = ['$http'];
    function MenuSearchService($http) {
        var service = this;
        var apiUrl = "https://davids-restaurant.herokuapp.com/menu_items.json";

        service.getMatchedMenuItems = function (searchTerm) {
            return $http.get(apiUrl).then(function (response) {
                var allItems = response.data.menu_items;
                var foundItems = allItems.filter(item => 
                    item.description.toLowerCase().includes(searchTerm.toLowerCase())
                );
                return foundItems;
            });
        };
    }

    function FoundItemsDirective() {
        return {
            restrict: 'E',
            templateUrl: 'foundItems.html',
            scope: {
                items: '<',
                onRemove: '&'
            }
        };
    }
})();
