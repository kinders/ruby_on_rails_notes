# vim: set ft=markdown :#
# 使用一个管理员角色管理用户

Based on Devise 1.1.3, Cancan 0.4.1, and uses Mongoid.
基于Devise 1.1.3,Cancan 0.4.1，使用Mongoid。

### routes.rb 路由

    DeviseRolesUserManagement::Application.routes.draw do
      devise_for :users
      devise_scope :user do
        get '/login' => 'devise/sessions#new'
        get '/logout' => 'devise/sessions#destroy'
      end
      resources :user, :controller => "user"
      root :to => "dashboard#index"
    end


### user_controller.rb 用户控制器
    class UserController < ApplicationController
      load_and_authorize_resource
  
      def index
        @users = User.excludes(:id => current_user.id)
      end
  
      def new
        @user = User.new
      end
  
      def create
        @user = User.new(params[:user])
        if @user.save
          flash[:notice] = "Successfully created User." 
          redirect_to root_path
        else
          render :action => 'new'
        end
      end
  
      def edit
        @user = User.find(params[:id])
      end
  
      def update
        @user = User.find(params[:id])
        params[:user].delete(:password) if params[:user][:password].blank?
        params[:user].delete(:password_confirmation) if params[:user][:password].blank? and params[:user][:password_confirmation].blank?
        if @user.update_attributes(params[:user])
          flash[:notice] = "Successfully updated User."
          redirect_to root_path
        else
          render :action => 'edit'
        end
      end
  
      def destroy
        @user = User.find(params[:id])
        if @user.destroy
          flash[:notice] = "Successfully deleted User."
          redirect_to root_path
        end
      end 
    end

The views provide the all the links for the Users with an admin role to add and manage Users. 
视图提供了所有的链接，用于具有管理员角色的用户添加和管理用户。
I just created a role field for  user and use CanCan to take care of the authorizations. 
我只创建了一个角色字段，使用CanCan来进行授权。

### ability.rb   能力模型

    class Ability
      include CanCan::Ability

      def initialize(user)
        can :manage, :all if user.role == "admin"
      end
    end

And then I catch unauthorized requests and redirect to root_url with a flash message.
然后捕捉非授权请求，重定向到根路径，带着一个闪存信息。

### application_controller.rb 程序控制器

    class ApplicationController < ActionController::Base
      protect_from_forgery
      rescue_from CanCan::AccessDenied do |exception|
        flash[:error] = exception.message
        redirect_to root_url
      end
    end



Brandon Martin
[Github App](http://github.com/brandmart/devise-roles-user-management)
