---
layout: post
title: BlocChat
thumbnail-path: "/img/blocChat/BlocChat_frontPage.png"
short-description: BlocChat is a messaging web application I built to learn the Angular framework and the fundamentals of Firebase

---

{:.center}
![]({{ site.baseurl }}/img/blocChat/BlocChat_frontPage.png)

## What is BlocChat?

BlocChat was the first solo project I set out on. It was based on an assignment to complete a messenger SPA during my coursework at Bloc. I wanted to create a messaging application that was clean, attractive to look at, and accomplished all of the basic user goals set out in front of me. 

## Where did BlocChat come from?

The assignment laid out in front of me was to create a single page messaging application using AngularJS to complete my Bloc front-end development module.  I was the sole developer on this project, with some limited guidance from my mentor and some assignment prompts from Bloc. 

## Problems

There were five key user stories set in front of me when I began BlocChat.
1. I wanted to create a list of available chat rooms.
2. I wanted to be able to create new chat rooms using a modal window.
3. I wanted to see a list of messages populate in each chat room once clicked. 
4. I wanted users to have to set a username upon loading the page,and I wanted the application to store a cookie on the user's browser to remember that username.
5. I wanted users to be able to send messages in different rooms that would be associated with their username. 

## Solutions

Creating a generic list of rooms was as simple as writing them up in Firebase and creating a Room factory service. 

![BlocChat sidebar roomlist](/img/blocChat/BlocChat_listAvailableRooms.png)

```javascript
function Room($firebaseArray) {
    var ref = firebase.database().ref().child("rooms");
    var rooms = $firebaseArray(ref);

    return {
        all: rooms,
        create: function(newRoom) {
            rooms.$add(newRoom);
```

I used UI-Bootstrap's $uib-modal method to create a modal that would allow users to create their own chat rooms. I added the button to the sidebar menu and created a pair of basic controllers to run the modal window.

![BlocChat room creator modal](/img/blocChat/BlocChat_createChatRoom.png)
![BlocChat full screen new Room modal](/img/blocChat/BlocChat_NewRoomModal.png)
```javascript
function ModalCtrl($uibModal, $log, Room) {
    var $ctrl = this;

    $ctrl.open = function() {
        var modalInstance = $uibModal.open({
        ariaLabelledBy: 'chat-modal-title',
        ariaDescribedBy: 'chat-modal-body',
        templateUrl: '/templates/chat_modal.html',
        controller: 'ModalInstanceCtrl',
        controllerAs: 'chatModal',
        keyboard: false,
        size: 'sm'

    });

    modalInstance.result.then(function(name) {
        $ctrl.room = name;
        Room.create($ctrl.room);
```

The next step was integrating the messaging system with the room database and figuring out how to load specialized messages for each room. For this I used a separate factory service. 

![BlocChat message display](/img/blocChat/BlocChat_displayMessages.png)

```javascript
function Message($firebaseArray, $cookies) {
    var Message = {};
    var ref = firebase.database().ref().child("messages");
    var messages = $firebaseArray(ref);
    var currentRoomId = null;

    Message.getByRoomId = function(roomId) {
    currentRoomId = roomId;
    Message.currentMessage = $firebaseArray(ref.orderByChild("roomId").equalTo(roomId));
```

Next, I wanted to have users create a username upon loading the application into their browser window. So I went back to the modal well, and created a new modal that would capture their username, and store it in a variable and onto the user's browser simultaneously. This time however, I created the modal within a runblock on my module, and made sure to set my keyboard value to false and backdrop to static so that users wouldn't be able to get around the modal.

![BlocChat username Modal](/img/blocChat/BlocChat_setUserName.png)

```javascript
function BlocChatCookies($cookies, $uibModal) {
    var $ctrl = this;
    var currentUser = $cookies.get('blocChatCurrentUser');
    if (!currentUser || currentUser === '') {
        var modalInstance = $uibModal.open({
        animation: $ctrl.animationsEnabled,
        templateUrl: '/templates/username_modal.html',
        controller: 'UsernameInstanceCtrl',
        controllerAs: 'usernameModal',
        ariaDescribedBy: 'username-modal-body',
        ariaLabelledBy: 'username-modal-title',
        keyboard: false,
        backdrop: 'static',
        size: 'sm',

    });

        $ctrl.toggleAnimation = function() {

        $ctrl.animationEnabled = !$ctrl.animationEnabled;
        }

        modalInstance.result.then(function(name) {
        this.username = name;
        $cookies.put('blocChatCurrentUser', name);
```

Finally, I wanted users to be able to click on an available room, and send messages associated with their created username to the selected room. For this I created a send method within my Message service that could control the sending of messages and add them to my firebase array. 

![blocChat open room final](/img/blocChat/BlocChat_openRoomFinal.png)
![blocChat send message window](/img/blocChat/BlocChat_sendMessage.png)

```javascript
Message.send = function() {
    if (currentRoomId) {
        messages.$add({
        username: $cookies.get('blocChatCurrentUser'),
        content: Message.messageString,
        roomId: currentRoomId,
        sentAt: convertTimeStamp(Date.now())
```

## Results

The application passed all requirements of the Bloc assignment with flying colors. Aside from some design choices I might reconsider now, I was very pleased with the final application's functionality. Once I was able to re-focus the work after getting a better understanding of some of the new tech I was working with,things clicked into place. Firebase in particular gave me some early issues, but I was able to work all of that out with some extra work on the documentation.

## Conclusion

The most important lesson I learned while crafting BlocChat was one that keeps reinforcing itself while I continue to learn, the simplest answer is usually the correct one. I found that while bells and whistles might look cool, it's all about the functionality. My issue early on was that I wanted a particular look for the application, and I mistakenly prioritized that over the necessariy functionality.  Once I re-organized myself and tore down my early work to focus on the inner workings of the application, things became much simpler and made more sense.

From this project I learned better troubleshooting skills, better application management and planning skills, and I gained a much better handle on data architechture in the AngularJS framework.

